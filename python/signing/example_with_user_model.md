Dưới đây là ví dụ triển khai **xác minh email khi đăng ký (email confirmation)** và **đặt lại mật khẩu (password reset)** bằng cách sử dụng `UserViewSet` trong **Django REST Framework (DRF)**, tích hợp module `django.core.signing`. Tôi sẽ giải thích bằng tiếng Việt, cung cấp mã nguồn chi tiết và áp dụng các phương pháp tốt nhất (best practices) đã đề cập trước đó.

---

## Tổng quan

- **Mục tiêu**: Tạo một `UserViewSet` để xử lý đăng ký người dùng, xác minh email, yêu cầu đặt lại mật khẩu và đặt lại mật khẩu.
- **Công cụ**: Sử dụng `TimestampSigner` để tạo và xác minh token, DRF để xây dựng API, và gửi email qua SMTP.
- **Cấu trúc**:
  - API đăng ký (`POST /users/register/`): Tạo người dùng mới và gửi email xác minh.
  - API xác minh email (`GET /users/verify-email/`): Kích hoạt tài khoản dựa trên token.
  - API yêu cầu đặt lại mật khẩu (`POST /users/password-reset-request/`): Gửi email chứa liên kết đặt lại mật khẩu.
  - API đặt lại mật khẩu (`POST /users/reset-password/`): Cập nhật mật khẩu mới dựa trên token.

---

## Cài đặt ban đầu

### Yêu cầu
- Đã cài đặt Django và Django REST Framework.
- Cấu hình email trong `settings.py` (ví dụ: sử dụng SMTP hoặc dịch vụ như SendGrid).
- Có model `User` tùy chỉnh (hoặc sử dụng `AbstractUser`).

### Cấu hình `settings.py`

```python
# settings.py
SECRET_KEY = "your-secret-key"  # Đảm bảo khóa bí mật an toàn
SITE_URL = "http://localhost:8000"  # URL của ứng dụng
DEFAULT_FROM_EMAIL = "no-reply@yourdomain.com"

# Cấu hình email (ví dụ: SMTP Gmail)
EMAIL_BACKEND = "django.core.mail.backends.smtp.EmailBackend"
EMAIL_HOST = "smtp.gmail.com"
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = "your-email@gmail.com"
EMAIL_HOST_PASSWORD = "your-app-password"
```

### Model người dùng

```python
# models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    email_verified = models.BooleanField(default=False)
```

Cập nhật `AUTH_USER_MODEL` trong `settings.py`:

```python
AUTH_USER_MODEL = "your_app.CustomUser"
```

---

## Triển khai `UserViewSet`

### Serializer

Tạo các serializer để xử lý dữ liệu đầu vào/đầu ra.

```python
# serializers.py
from rest_framework import serializers
from django.contrib.auth import get_user_model

User = get_user_model()

class UserRegisterSerializer(serializers.ModelSerializer):
    password = serializers.CharField(write_only=True, min_length=8)

    class Meta:
        model = User
        fields = ["username", "email", "password"]

    def create(self, validated_data):
        user = User.objects.create_user(
            username=validated_data["username"],
            email=validated_data["email"],
            password=validated_data["password"],
            is_active=False,  # Chưa kích hoạt
            email_verified=False,
        )
        return user

class PasswordResetRequestSerializer(serializers.Serializer):
    email = serializers.EmailField()

class PasswordResetSerializer(serializers.Serializer):
    token = serializers.CharField()
    password = serializers.CharField(min_length=8)
    password_confirm = serializers.CharField(min_length=8)

    def validate(self, data):
        if data["password"] != data["password_confirm"]:
            raise serializers.ValidationError("Mật khẩu xác nhận không khớp.")
        return data
```

### Hàm hỗ trợ gửi email

Tạo các hàm để gửi email xác minh và đặt lại mật khẩu.

```python
# utils.py
from django.core.signing import TimestampSigner
from django.urls import reverse
from django.utils.http import urlsafe_base64_encode, urlsafe_base64_decode
from django.utils.encoding import force_bytes, force_str
from django.core.mail import send_mail
from django.conf import settings

def send_verification_email(user):
    signer = TimestampSigner()
    value = str(user.id)
    signed_value = signer.sign(value)
    token = urlsafe_base64_encode(force_bytes(signed_value))
    
    verification_url = (
        f"{settings.SITE_URL}"
        f"{reverse('user-viewset-verify-email')}?token={token}"
    )
    
    send_mail(
        subject="Xác minh tài khoản của bạn",
        message=f"Vui lòng nhấp vào liên kết để xác minh email: {verification_url}",
        from_email=settings.DEFAULT_FROM_EMAIL,
        recipient_list=[user.email],
        fail_silently=False,
    )

def send_password_reset_email(user):
    signer = TimestampSigner()
    value = str(user.id)
    signed_value = signer.sign(value)
    token = urlsafe_base64_encode(force_bytes(signed_value))
    
    reset_url = (
        f"{settings.SITE_URL}"
        f"{reverse('user-viewset-reset-password')}?token={token}"
    )
    
    send_mail(
        subject="Đặt lại mật khẩu",
        message=f"Nhấp vào liên kết để đặt lại mật khẩu: {reset_url}",
        from_email=settings.DEFAULT_FROM_EMAIL,
        recipient_list=[user.email],
        fail_silently=False,
    )
```

### UserViewSet

Tạo `UserViewSet` để xử lý các hành động liên quan.

```python
# views.py
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from django.contrib.auth import get_user_model
from django.core.signing import TimestampSigner, BadSignature, SignatureExpired
from django.utils.http import urlsafe_base64_decode
from django.utils.encoding import force_str
from .serializers import UserRegisterSerializer, PasswordResetRequestSerializer, PasswordResetSerializer
from .utils import send_verification_email, send_password_reset_email

User = get_user_model()

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserRegisterSerializer

    @action(detail=False, methods=["post"], url_path="register")
    def register(self, request):
        """Đăng ký người dùng mới và gửi email xác minh."""
        serializer = UserRegisterSerializer(data=request.data)
        if serializer.is_valid():
            user = serializer.save()
            send_verification_email(user)
            return Response(
                {"message": "Đăng ký thành công! Vui lòng kiểm tra email để xác minh."},
                status=status.HTTP_201_CREATED
            )
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    @action(detail=False, methods=["get"], url_path="verify-email")
    def verify_email(self, request):
        """Xác minh email dựa trên token."""
        token = request.query_params.get("token")
        if not token:
            return Response(
                {"error": "Token không hợp lệ!"},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        signer = TimestampSigner()
        try:
            signed_value = force_str(urlsafe_base64_decode(token))
            user_id = signer.unsign(signed_value, max_age=24*3600)  # Hết hạn sau 24 giờ
            user = User.objects.get(id=user_id)
            
            if not user.email_verified:
                user.email_verified = True
                user.is_active = True
                user.save()
                return Response(
                    {"message": "Email đã được xác minh thành công!"},
                    status=status.HTTP_200_OK
                )
            return Response(
                {"message": "Email đã được xác minh trước đó!"},
                status=status.HTTP_200_OK
            )
            
        except (BadSignature, SignatureExpired, User.DoesNotExist):
            return Response(
                {"error": "Token không hợp lệ hoặc đã hết hạn!"},
                status=status.HTTP_400_BAD_REQUEST
            )

    @action(detail=False, methods=["post"], url_path="password-reset-request")
    def password_reset_request(self, request):
        """Yêu cầu đặt lại mật khẩu và gửi email."""
        serializer = PasswordResetRequestSerializer(data=request.data)
        if serializer.is_valid():
            email = serializer.validated_data["email"]
            try:
                user = User.objects.get(email=email)
                send_password_reset_email(user)
                return Response(
                    {"message": "Vui lòng kiểm tra email để đặt lại mật khẩu!"},
                    status=status.HTTP_200_OK
                )
            except User.DoesNotExist:
                # Không tiết lộ email không tồn tại
                return Response(
                    {"message": "Vui lòng kiểm tra email để đặt lại mật khẩu!"},
                    status=status.HTTP_200_OK
                )
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    @action(detail=False, methods=["post"], url_path="reset-password")
    def reset_password(self, request):
        """Đặt lại mật khẩu dựa trên token."""
        serializer = PasswordResetSerializer(data=request.data)
        if serializer.is_valid():
            token = serializer.validated_data["token"]
            password = serializer.validated_data["password"]
            
            signer = TimestampSigner()
            try:
                signed_value = force_str(urlsafe_base64_decode(token))
                user_id = signer.unsign(signed_value, max_age=3600)  # Hết hạn sau 1 giờ
                user = User.objects.get(id=user_id)
                
                user.set_password(password)
                user.save()
                return Response(
                    {"message": "Mật khẩu đã được đặt lại thành công!"},
                    status=status.HTTP_200_OK
                )
                
            except (BadSignature, SignatureExpired, User.DoesNotExist):
                return Response(
                    {"error": "Token không hợp lệ hoặc đã hết hạn!"},
                    status=status.HTTP_400_BAD_REQUEST
                )
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

### URL

Cấu hình URL để truy cập các endpoint.

```python
# urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import UserViewSet

router = DefaultRouter()
router.register(r"users", UserViewSet, basename="user-viewset")

urlpatterns = [
    path("", include(router.urls)),
]
```

---

## Cách sử dụng API

### 1. Đăng ký người dùng

- **Endpoint**: `POST /users/register/`
- **Body**:
  ```json
  {
    "username": "nguyen",
    "email": "nguyen@example.com",
    "password": "securepassword123"
  }
  ```
- **Phản hồi**:
  ```json
  {
    "message": "Đăng ký thành công! Vui lòng kiểm tra email để xác minh."
  }
  ```
- **Hành động**: Một email chứa liên kết xác minh sẽ được gửi đến `nguyen@example.com`.

### 2. Xác minh email

- **Endpoint**: `GET /users/verify-email/?token=<encoded_token>`
- **Phản hồi** (thành công):
  ```json
  {
    "message": "Email đã được xác minh thành công!"
  }
  ```
- **Phản hồi** (lỗi):
  ```json
  {
    "error": "Token không hợp lệ hoặc đã hết hạn!"
  }
  ```

### 3. Yêu cầu đặt lại mật khẩu

- **Endpoint**: `POST /users/password-reset-request/`
- **Body**:
  ```json
  {
    "email": "nguyen@example.com"
  }
  ```
- **Phản hồi**:
  ```json
  {
    "message": "Vui lòng kiểm tra email để đặt lại mật khẩu!"
  }
  ```
- **Hành động**: Một email chứa liên kết đặt lại mật khẩu sẽ được gửi.

### 4. Đặt lại mật khẩu

- **Endpoint**: `POST /users/reset-password/`
- **Body**:
  ```json
  {
    "token": "<encoded_token>",
    "password": "newpassword123",
    "password_confirm": "newpassword123"
  }
  ```
- **Phản hồi** (thành công):
  ```json
  {
    "message": "Mật khẩu đã được đặt lại thành công!"
  }
  ```
- **Phản hồi** (lỗi):
  ```json
  {
    "error": "Token không hợp lệ hoặc đã hết hạn!"
  }
  ```

---

## Phương pháp tốt nhất được áp dụng

1. **Bảo mật token**:
   - Sử dụng `TimestampSigner` để tạo token có thời hạn (24 giờ cho xác minh email, 1 giờ cho đặt lại mật khẩu).
   - Mã hóa token bằng `urlsafe_base64_encode` để an toàn trong URL.

2. **Xử lý lỗi an toàn**:
   - Không tiết lộ thông tin nhạy cảm (ví dụ: email không tồn tại trong password reset).
   - Trả về thông báo lỗi chung cho các trường hợp token không hợp lệ hoặc hết hạn.

3. **Tối ưu hiệu suất**:
   - Sử dụng serializer để kiểm tra dữ liệu đầu vào.
   - Có thể tích hợp Celery để gửi email bất đồng bộ (nếu cần).

4. **Bảo mật email**:
   - Sử dụng SMTP với TLS/SSL.
   - Không bao gồm thông tin nhạy cảm trong nội dung email.

5. **Kiểm tra trạng thái**:
   - Kiểm tra `email_verified` để tránh xác minh lại.
   - Đảm bảo người dùng tồn tại trước khi xử lý token.

---

## Mở rộng (Tùy chọn)

1. **Token dùng một lần**:
   - Lưu token trong cơ sở dữ liệu hoặc Redis và vô hiệu hóa sau khi sử dụng.
   - Ví dụ: Thêm model `PasswordResetToken` để theo dõi trạng thái.

2. **Gửi email bất đồng bộ**:
   - Sử dụng Celery để gửi email:
     ```python
     from celery import shared_task

     @shared_task
     def send_verification_email_async(user_id):
         user = User.objects.get(id=user_id)
         send_verification_email(user)
     ```

3. **Rate limiting**:
   - Hạn chế số lần yêu cầu đặt lại mật khẩu hoặc gửi email xác minh để tránh lạm dụng, sử dụng `django-ratelimit` hoặc DRF throttling.

---

## Kết luận

`UserViewSet` trên cung cấp một cách triển khai hoàn chỉnh và an toàn cho email confirmation và password reset bằng `django.core.signing` trong DRF. Các endpoint được thiết kế để dễ sử dụng, bảo mật và tuân theo các phương pháp tốt nhất. Nếu bạn cần thêm tính năng (như tích hợp Celery, rate limiting) hoặc giải thích chi tiết hơn, hãy cho tôi biết!