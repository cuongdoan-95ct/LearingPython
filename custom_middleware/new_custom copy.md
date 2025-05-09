Để tùy chỉnh cấu trúc của các ngoại lệ (API exceptions) trả về cho frontend (FE) trong Django REST Framework (DRF), bao gồm cả các ngoại lệ được bắt ở middleware, bạn cần thực hiện hai bước chính:
1. **Tùy chỉnh trình xử lý ngoại lệ của DRF** để định dạng lại các lỗi từ serializer và các `APIException` khác.
2. **Cập nhật middleware** để đảm bảo các ngoại lệ không được DRF xử lý (như `ValueError`, `PermissionDenied`, v.v.) cũng tuân theo cùng một cấu trúc lỗi.

Dưới đây là cách triển khai chi tiết để đảm bảo tất cả các phản hồi lỗi (từ DRF và middleware) có cùng cấu trúc JSON gửi về FE, cùng với ví dụ cụ thể.

---

### Cấu trúc lỗi mong muốn
Giả sử bạn muốn tất cả các phản hồi lỗi gửi về FE có cấu trúc JSON thống nhất như sau:
```json
{
  "status": "error",
  "error_code": "<mã lỗi cụ thể>",
  "message": "<thông điệp lỗi>",
  "details": "<chi tiết lỗi, có thể là object hoặc string>",
  "timestamp": "<thời gian xảy ra lỗi, định dạng ISO>"
}
```

---

### 1. Tùy chỉnh trình xử lý ngoại lệ của DRF
DRF cho phép bạn ghi đè trình xử lý ngoại lệ mặc định thông qua cài đặt `EXCEPTION_HANDLER`. Bạn sẽ tạo một hàm xử lý ngoại lệ tùy chỉnh để định dạng lại các lỗi từ serializer và các `APIException`.

#### Bước 1: Tạo hàm xử lý ngoại lệ tùy chỉnh
Tạo một tệp `exceptions.py` trong ứng dụng của bạn (ví dụ: `myapp/exceptions.py`).

```python
from rest_framework.views import exception_handler
from rest_framework.exceptions import APIException
from django.http import JsonResponse
from datetime import datetime
import pytz

def custom_exception_handler(exc, context):
    # Gọi trình xử lý ngoại lệ mặc định của DRF
    response = exception_handler(exc, context)

    # Định dạng phản hồi lỗi
    error_response = {
        "status": "error",
        "error_code": getattr(exc, "default_code", "unknown_error"),
        "message": str(exc),
        "details": None,
        "timestamp": datetime.now(pytz.UTC).isoformat(),
    }

    if response is not None:
        # Xử lý lỗi từ DRF (APIException, ValidationError, v.v.)
        error_response["error_code"] = getattr(exc, "default_code", response.status_code)
        error_response["message"] = response.data.get("detail", str(exc))
        error_response["details"] = response.data if isinstance(response.data, dict) else str(response.data)
        response.data = error_response
    else:
        # Nếu không có response (lỗi không phải APIException), để middleware xử lý
        return None

    return response
```

#### Bước 2: Cấu hình DRF để sử dụng trình xử lý ngoại lệ tùy chỉnh
Thêm cấu hình vào `settings.py`:

```python
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'myapp.exceptions.custom_exception_handler',
}
```

#### Giải thích:
- **`custom_exception_handler`**:
  - Gọi `exception_handler` mặc định của DRF để xử lý các lỗi như `ValidationError`, `PermissionDenied`, v.v.
  - Tạo một `error_response` với cấu trúc mong muốn.
  - Nếu DRF xử lý lỗi (có `response`), định dạng lại dữ liệu lỗi thành cấu trúc thống nhất.
  - Nếu không có `response` (lỗi không phải `APIException`), trả về `None` để middleware xử lý.
- **Các trường**:
  - `status`: Luôn là `"error"`.
  - `error_code`: Lấy từ `default_code` của ngoại lệ hoặc mã trạng thái HTTP.
  - `message`: Thông điệp lỗi, ưu tiên `detail` từ DRF.
  - `details`: Chi tiết lỗi, có thể là dictionary (từ `ValidationError`) hoặc string.
  - `timestamp`: Thời gian lỗi xảy ra, định dạng ISO.

#### Ví dụ phản hồi từ Serializer:
Nếu serializer ném `ValidationError`:
```python
from rest_framework import serializers

class UserSerializer(serializers.Serializer):
    username = serializers.CharField(max_length=100)

    def validate_username(self, value):
        if len(value) < 3:
            raise serializers.ValidationError("Username must be at least 3 characters long.")
        return value
```
Gửi dữ liệu không hợp lệ:
```json
{"username": "ab"}
```
Phản hồi:
```json
{
  "status": "error",
  "error_code": "invalid",
  "message": "Username must be at least 3 characters long.",
  "details": {
    "username": ["Username must be at least 3 characters long."]
  },
  "timestamp": "2025-05-09T12:34:56.789123+00:00"
}
```

---

### 2. Cập nhật Middleware để xử lý các ngoại lệ không phải APIException
Middleware sẽ xử lý các ngoại lệ không được DRF bắt (như `ValueError`, `Exception`, v.v.) và đảm bảo phản hồi lỗi có cùng cấu trúc như trên.

#### Bước 3: Cập nhật CustomErrorMiddleware
Cập nhật middleware để trả về phản hồi lỗi với cấu trúc thống nhất.

```python
from django.http import JsonResponse
from django.core.exceptions import PermissionDenied
import logging
from datetime import datetime
import pytz

class CustomErrorMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        response = self.get_response(request)
        return response

    def process_exception(self, request, exception):
        # Ghi log lỗi
        logger = logging.getLogger(__name__)
        logger.error(f"Error occurred: {str(exception)}", exc_info=True)

        # Định dạng phản hồi lỗi
        error_response = {
            "status": "error",
            "error_code": "server_error",
            "message": str(exception),
            "details": None,
            "timestamp": datetime.now(pytz.UTC).isoformat(),
        }

        # Xử lý các loại ngoại lệ cụ thể
        if isinstance(exception, ValueError):
            error_response["error_code"] = "invalid_input"
            error_response["message"] = "Invalid input provided."
            error_response["details"] = str(exception)
            status_code = 400
        elif isinstance(exception, PermissionDenied):
            error_response["error_code"] = "permission_denied"
            error_response["message"] = "Permission denied."
            error_response["details"] = str(exception)
            status_code = 403
        else:
            status_code = 500

        return JsonResponse(error_response, status=status_code)
```

#### Giải thích:
- **Cấu trúc lỗi**: Giống hệt cấu trúc trong `custom_exception_handler` để đảm bảo thống nhất.
- **Xử lý ngoại lệ**:
  - `ValueError`: Mã lỗi `invalid_input`, HTTP 400.
  - `PermissionDenied`: Mã lỗi `permission_denied`, HTTP 403.
  - Lỗi chung: Mã lỗi `server_error`, HTTP 500.
- **Log**: Ghi lại lỗi để dễ debug.

#### Ví dụ phản hồi từ Middleware:
Nếu view ném `ValueError`:
```python
from rest_framework.views import APIView
from rest_framework.response import Response

class TestView(APIView):
    def get(self, request):
        raise ValueError("Invalid data")
        return Response({"message": "Success"})
```
Phản hồi:
```json
{
  "status": "error",
  "error_code": "invalid_input",
  "message": "Invalid input provided.",
  "details": "Invalid data",
  "timestamp": "2025-05-09T12:34:56.789123+00:00"
}
```

---

### 3. Kiểm tra cấu trúc lỗi
Để đảm bảo cả DRF và middleware trả về đúng cấu trúc lỗi, bạn cần viết các bài kiểm tra.

#### Kiểm tra Serializer (DRF Exception)
```python
from django.test import TestCase
from rest_framework.test import APIClient
from rest_framework import serializers
import json

class UserSerializer(serializers.Serializer):
    username = serializers.CharField(max_length=100)

    def validate_username(self, value):
        if len(value) < 3:
            raise serializers.ValidationError("Username must be at least 3 characters long.")
        return value

class TestAPIExceptions(TestCase):
    def setUp(self):
        self.client = APIClient()

    def test_serializer_validation_error(self):
        data = {"username": "ab"}
        response = self.client.post('/api/test/', data, format='json')
        self.assertEqual(response.status_code, 400)
        data = json.loads(response.content)
        self.assertEqual(data['status'], 'error')
        self.assertEqual(data['error_code'], 'invalid')
        self.assertEqual(data['message'], 'Username must be at least 3 characters long.')
        self.assertTrue('details' in data)
        self.assertTrue('timestamp' in data)
```

#### Kiểm tra Middleware
```python
from django.test import TestCase
from django.urls import path
from django.http import JsonResponse
from django.core.exceptions import PermissionDenied
import json

# View giả lập
def test_value_error(request):
    raise ValueError("Invalid data")

def test_permission_error(request):
    raise PermissionDenied("Access denied")

urlpatterns = [
    path('test/value-error/', test_value_error),
    path('test/permission-error/', test_permission_error),
]

class CustomErrorMiddlewareTestCase(TestCase):
    def setUp(self):
        self.client = Client()

    def test_value_error_response(self):
        response = self.client.get('/test/value-error/')
        self.assertEqual(response.status_code, 400)
        data = json.loads(response.content)
        self.assertEqual(data['status'], 'error')
        self.assertEqual(data['error_code'], 'invalid_input')
        self.assertEqual(data['message'], 'Invalid input provided.')
        self.assertEqual(data['details'], 'Invalid data')
        self.assertTrue('timestamp' in data)

    def test_permission_error_response(self):
        response = self.client.get('/test/permission-error/')
        self.assertEqual(response.status_code, 403)
        data = json.loads(response.content)
        self.assertEqual(data['status'], 'error')
        self.assertEqual(data['error_code'], 'permission_denied')
        self.assertEqual(data['message'], 'Permission denied.')
        self.assertEqual(data['details'], 'Access denied')
        self.assertTrue('timestamp' in data)
```

#### Cách chạy kiểm tra:
1. Lưu các tệp kiểm tra vào thư mục `tests` của ứng dụng.
2. Chạy:
   ```bash
   python manage.py test
   ```
3. Đảm bảo tất cả kiểm tra pass, xác nhận rằng cả DRF và middleware trả về đúng cấu trúc lỗi.

---

### 4. Kiểm tra thủ công
Để kiểm tra thủ công:
1. Chạy server:
   ```bash
   python manage.py runserver
   ```
2. Gửi yêu cầu đến endpoint:
   - **Serializer**: POST dữ liệu không hợp lệ đến endpoint sử dụng serializer.
     ```bash
     curl -X POST -H "Content-Type: application/json" -d '{"username": "ab"}' http://127.0.0.1:8000/api/test/
     ```
   - **Middleware**: Gửi GET đến endpoint ném `ValueError`.
     ```bash
     curl -i http://127.0.0.1:8000/test/value-error/
     ```
3. Kiểm tra phản hồi JSON có đúng cấu trúc:
   - `status`, `error_code`, `message`, `details`, `timestamp`.

---

### 5. Lưu ý và Tùy chỉnh thêm
- **Thêm mã lỗi cụ thể**: Bạn có thể mở rộng `error_code` bằng cách tạo danh sách mã lỗi tùy chỉnh (ví dụ: `INVALID_USERNAME`, `AUTH_FAILED`).
- **Xử lý lỗi khác**: Thêm logic trong `custom_exception_handler` hoặc middleware để xử lý các ngoại lệ như `Http404`, `AuthenticationFailed`, v.v.
- **Đa ngôn ngữ**: Nếu FE yêu cầu thông điệp lỗi đa ngôn ngữ, tích hợp `django.utils.translation` để dịch `message`.
- **Log chi tiết**: Cấu hình logging trong `settings.py` để ghi lại toàn bộ chi tiết lỗi.

#### Ví dụ cấu hình logging:
```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'ERROR',
            'class': 'logging.FileHandler',
            'filename': 'errors.log',
        },
    },
    'loggers': {
        '': {
            'handlers': ['file'],
            'level': 'ERROR',
            'propagate': True,
        },
    },
}
```

Nếu bạn cần thêm tính năng cụ thể (như xử lý lỗi cho một ngoại lệ đặc biệt hoặc tích hợp với FE cụ thể), hãy cho tôi biết!