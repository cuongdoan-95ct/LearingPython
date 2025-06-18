# Tạo Validators cho Django Model Field (Django 5.1 + DRF)

Để tạo validators có thể sử dụng đồng thời cho form, serializer (DRF) và shell trong Django 5.1, bạn có thể áp dụng cách tiếp cận sau:

## 1. Tạo Custom Validators

Tạo file `validators.py` trong app của bạn:

```python
from django.core.exceptions import ValidationError
from django.utils.translation import gettext_lazy as _

def validate_username(value):
    """
    Validator cho username
    - Không chứa ký tự đặc biệt
    - Không chứa khoảng trắng
    - Độ dài tối thiểu 4 ký tự
    """
    if len(value) < 4:
        raise ValidationError(
            _('Username phải có ít nhất 4 ký tự'),
            code='username_too_short'
        )
    
    if not value.isalnum():
        raise ValidationError(
            _('Username chỉ được chứa chữ cái và số'),
            code='username_invalid_chars'
        )

def validate_password_strength(value):
    """
    Validator cho password
    - Ít nhất 8 ký tự
    - Chứa ít nhất 1 chữ hoa, 1 chữ thường, 1 số
    """
    if len(value) < 8:
        raise ValidationError(
            _('Mật khẩu phải có ít nhất 8 ký tự'),
            code='password_too_short'
        )
    
    if not any(char.isdigit() for char in value):
        raise ValidationError(
            _('Mật khẩu phải chứa ít nhất 1 số'),
            code='password_no_number'
        )
    
    if not any(char.isupper() for char in value):
        raise ValidationError(
            _('Mật khẩu phải chứa ít nhất 1 chữ hoa'),
            code='password_no_upper'
        )
    
    if not any(char.islower() for char in value):
        raise ValidationError(
            _('Mật khẩu phải chứa ít nhất 1 chữ thường'),
            code='password_no_lower'
        )
```

## 2. Áp dụng vào Model

Trong file `models.py`:

```python
from django.db import models
from django.contrib.auth.models import AbstractUser
from .validators import validate_username, validate_password_strength

class User(AbstractUser):
    username = models.CharField(
        max_length=150,
        unique=True,
        validators=[validate_username],
        error_messages={
            'unique': _("Username này đã được sử dụng."),
        },
    )
    
    password = models.CharField(
        max_length=128,
        validators=[validate_password_strength]
    )
    
    email = models.EmailField(unique=True)
    
    def clean(self):
        super().clean()
        # Thêm validation mức model nếu cần
        if self.username.lower() == 'admin':
            raise ValidationError(
                {'username': _("Không được sử dụng username 'admin'")}
            )
    
    def save(self, *args, **kwargs):
        self.full_clean()  # Kích hoạt validation khi save
        super().save(*args, **kwargs)
```

## 3. Tạo Serializer cho DRF

Trong file `serializers.py`:

```python
from rest_framework import serializers
from django.contrib.auth.password_validation import validate_password
from .models import User
from .validators import validate_username, validate_password_strength

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'password']
        extra_kwargs = {
            'password': {'write_only': True},
            'username': {'validators': [validate_username]},
        }
    
    def validate_password(self, value):
        validate_password_strength(value)
        return value
    
    def create(self, validated_data):
        user = User.objects.create_user(
            username=validated_data['username'],
            email=validated_data['email'],
            password=validated_data['password']
        )
        return user
```

## 4. Tạo Form cho Django Admin/Views

Trong file `forms.py`:

```python
from django import forms
from django.contrib.auth.forms import UserCreationForm
from .models import User
from .validators import validate_username, validate_password_strength

class UserCreationForm(UserCreationForm):
    class Meta:
        model = User
        fields = ('username', 'email', 'password1', 'password2')
    
    def clean_username(self):
        username = self.cleaned_data['username']
        validate_username(username)
        return username
    
    def clean_password1(self):
        password1 = self.cleaned_data.get('password1')
        if password1:
            validate_password_strength(password1)
        return password1
```

## 5. Sử dụng trong Shell

Khi sử dụng trong shell, các validator sẽ tự động được kích hoạt:

```python
from myapp.models import User
from django.core.exceptions import ValidationError

try:
    user = User(username='abc', email='test@example.com', password='weak')
    user.full_clean()  # Kích hoạt validation
    user.save()
except ValidationError as e:
    print("Lỗi validation:", e.message_dict)
```

## 6. Kiểm tra tính nhất quán

Để đảm bảo validator hoạt động nhất quán trên tất cả các lớp (model, form, serializer):

```python
# Trong file tests/test_validators.py
from django.test import TestCase
from django.core.exceptions import ValidationError
from myapp.models import User
from myapp.forms import UserCreationForm
from myapp.serializers import UserSerializer
from rest_framework.exceptions import ValidationError as DRFValidationError

class ValidatorConsistencyTest(TestCase):
    def test_username_validation(self):
        # Test model
        with self.assertRaises(ValidationError):
            user = User(username='a b', email='test@example.com', password='ValidPass123')
            user.full_clean()
        
        # Test form
        form = UserCreationForm(data={'username': 'a b', 'password1': 'ValidPass123', 'password2': 'ValidPass123'})
        self.assertFalse(form.is_valid())
        
        # Test serializer
        serializer = UserSerializer(data={'username': 'a b', 'password': 'ValidPass123'})
        with self.assertRaises(DRFValidationError):
            serializer.is_valid(raise_exception=True)
```

## Lưu ý quan trọng

1. Luôn sử dụng `full_clean()` hoặc `clean()` khi save model trong shell
2. Đối với DRF, serializer sẽ tự động chạy validation khi gọi `is_valid()`
3. Đối với form, validation sẽ chạy khi gọi `is_valid()`
4. Sử dụng cùng một bộ validator cho tất cả các lớp để đảm bảo tính nhất quán
5. Cân nhắc sử dụng `gettext_lazy` (_) cho các message để hỗ trợ đa ngôn ngữ

Cách tiếp cận này đảm bảo validation thống nhất trên tất cả các lớp trong ứng dụng của bạn.