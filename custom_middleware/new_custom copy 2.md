Trong Django REST Framework (DRF), các lỗi như **invalid HTTP method** (phương thức HTTP không được phép, ví dụ: sử dụng POST cho một endpoint chỉ hỗ trợ GET) và **permission errors** (lỗi quyền truy cập, ví dụ: người dùng không có quyền thực hiện hành động) được xử lý ở các tầng khác nhau của framework. Dưới đây là phân tích chi tiết về nơi các lỗi này được bắt, kèm theo cách chúng được xử lý trong ngữ cảnh bạn đã yêu cầu (tùy chỉnh cấu trúc lỗi gửi về frontend, bao gồm cả middleware).

---

### 1. Lỗi Invalid HTTP Method
Lỗi invalid HTTP method xảy ra khi một yêu cầu sử dụng phương thức HTTP không được hỗ trợ bởi endpoint (ví dụ: gửi POST đến một view chỉ cho phép GET).

#### Nơi lỗi được bắt:
- **DRF View**: DRF tự động kiểm tra phương thức HTTP dựa trên các phương thức được định nghĩa trong view (ví dụ: `get`, `post`, `put`, v.v.). Nếu phương thức không được hỗ trợ, DRF ném ra ngoại lệ `MethodNotAllowed` (một loại `APIException`).
- **Tầng xử lý**: Lỗi này được bắt bởi **trình xử lý ngoại lệ của DRF** (`exception_handler`) trước khi đến middleware, vì `MethodNotAllowed` là một `APIException`.

#### Cách lỗi được xử lý:
- Mặc định, DRF trả về phản hồi lỗi với mã trạng thái HTTP **405 Method Not Allowed** và thông điệp chi tiết.
- Nếu bạn đã tùy chỉnh trình xử lý ngoại lệ của DRF (như trong `custom_exception_handler` đã cung cấp), lỗi `MethodNotAllowed` sẽ được định dạng lại theo cấu trúc JSON tùy chỉnh của bạn.

#### Ví dụ:
Giả sử bạn có một view chỉ hỗ trợ GET:
```python
from rest_framework.views import APIView
from rest_framework.response import Response

class TestView(APIView):
    def get(self, request):
        return Response({"message": "Success"})
```

Gửi yêu cầu POST đến endpoint này:
```bash
curl -X POST http://127.0.0.1:8000/test/
```

**Phản hồi mặc định của DRF** (nếu không tùy chỉnh):
```json
{
  "detail": "Method \"POST\" not allowed."
}
```

**Phản hồi với `custom_exception_handler`** (dựa trên cấu trúc đã tùy chỉnh trước đó):
```json
{
  "status": "error",
  "error_code": "method_not_allowed",
  "message": "Method \"POST\" not allowed.",
  "details": "Method \"POST\" not allowed.",
  "timestamp": "2025-05-09T12:34:56.789123+00:00"
}
```

#### Nơi bắt:
- Lỗi `MethodNotAllowed` được bắt trong **DRF view layer** và xử lý bởi **DRF exception handler**. Nó **không** đi đến middleware trừ khi bạn cố ý bỏ qua xử lý của DRF (rất hiếm).

---

### 2. Lỗi Permission (Permission Denied)
Lỗi permission xảy ra khi người dùng không có quyền thực hiện hành động (ví dụ: truy cập endpoint yêu cầu xác thực hoặc vai trò cụ thể).

#### Nơi lỗi được bắt:
- **DRF Permission Classes**: DRF kiểm tra quyền truy cập thông qua các lớp quyền (`permission_classes`) được định nghĩa trong view. Nếu quyền bị từ chối, DRF ném ngoại lệ `PermissionDenied` (một loại `APIException`).
- **Tầng xử lý**: Tương tự như `MethodNotAllowed`, lỗi `PermissionDenied` được bắt bởi **trình xử lý ngoại lệ của DRF** (`exception_handler`) trước khi đến middleware.
- **Trường hợp đặc biệt**: Nếu lỗi `PermissionDenied` được ném ngoài DRF (ví dụ: trong một view Django thông thường hoặc logic tùy chỉnh không sử dụng DRF), nó sẽ được xử lý bởi **middleware**.

#### Cách lỗi được xử lý:
- **Trong DRF**: Lỗi `PermissionDenied` trả về mã trạng thái HTTP **403 Forbidden** với thông điệp chi tiết. Nếu bạn đã tùy chỉnh `custom_exception_handler`, lỗi sẽ được định dạng lại theo cấu trúc JSON của bạn.
- **Trong Middleware**: Nếu lỗi `PermissionDenied` không được DRF xử lý (ví dụ: ném từ một view không phải DRF), middleware sẽ bắt và định dạng phản hồi (như trong `CustomErrorMiddleware` đã cung cấp).

#### Ví dụ:
Giả sử bạn có một view yêu cầu người dùng đã xác thực:
```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated

class SecureView(APIView):
    permission_classes = [IsAuthenticated]

    def get(self, request):
        return Response({"message": "Success"})
```

Gửi yêu cầu mà không có token xác thực:
```bash
curl -X GET http://127.0.0.1:8000/secure/
```

**Phản hồi mặc định của DRF**:
```json
{
  "detail": "Authentication credentials were not provided."
}
```

**Phản hồi với `custom_exception_handler`**:
```json
{
  "status": "error",
  "error_code": "permission_denied",
  "message": "Authentication credentials were not provided.",
  "details": "Authentication credentials were not provided.",
  "timestamp": "2025-05-09T12:34:56.789123+00:00"
}
```

#### Trường hợp PermissionDenied trong view không phải DRF:
Nếu bạn ném `PermissionDenied` trong một view Django thông thường:
```python
from django.core.exceptions import PermissionDenied
from django.http import HttpResponse

def test_view(request):
    raise PermissionDenied("Access denied")
    return HttpResponse("Success")
```

Lỗi này sẽ được **middleware** bắt (chứ không phải DRF), và với `CustomErrorMiddleware` đã tùy chỉnh, phản hồi sẽ là:
```json
{
  "status": "error",
  "error_code": "permission_denied",
  "message": "Permission denied.",
  "details": "Access denied",
  "timestamp": "2025-05-09T12:34:56.789123+00:00"
}
```

#### Nơi bắt:
- **Trong DRF**: Lỗi `PermissionDenied` được bắt bởi **DRF permission layer** và xử lý bởi **DRF exception handler** (HTTP 403).
- **Ngoài DRF**: Lỗi `PermissionDenied` được bắt bởi **middleware** (HTTP 403).

---

### 3. Tóm tắt nơi bắt và xử lý
| Loại lỗi                     | Nơi bắt                              | Tầng xử lý chính                     | Mã trạng thái |
|------------------------------|--------------------------------------|--------------------------------------|---------------|
| **Invalid HTTP Method**      | DRF View (`MethodNotAllowed`)        | DRF Exception Handler (`exception_handler`) | 405           |
| **Permission Denied (DRF)**  | DRF Permission Classes (`PermissionDenied`) | DRF Exception Handler (`exception_handler`) | 403           |
| **Permission Denied (Non-DRF)** | View hoặc logic tùy chỉnh (`PermissionDenied`) | Middleware (`process_exception`) | 403           |

---

### 4. Đảm bảo cấu trúc lỗi thống nhất
Để đảm bảo cả lỗi `MethodNotAllowed` và `PermissionDenied` (từ DRF và non-DRF) có cùng cấu trúc JSON, bạn đã có:
- **`custom_exception_handler`** (xử lý lỗi DRF, bao gồm `MethodNotAllowed` và `PermissionDenied` từ DRF).
- **`CustomErrorMiddleware`** (xử lý lỗi non-DRF, như `PermissionDenied` từ view thông thường).

Cả hai đều sử dụng cấu trúc JSON:
```json
{
  "status": "error",
  "error_code": "<mã lỗi>",
  "message": "<thông điệp>",
  "details": "<chi tiết>",
  "timestamp": "<thời gian ISO>"
}
```

#### Cập nhật `custom_exception_handler` để xử lý `MethodNotAllowed` rõ ràng
Nếu bạn muốn tùy chỉnh thêm cho `MethodNotAllowed`, cập nhật `exceptions.py`:

```python
from rest_framework.views import exception_handler
from rest_framework.exceptions import APIException, MethodNotAllowed
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
        # Xử lý lỗi từ DRF
        error_response["error_code"] = getattr(exc, "default_code", response.status_code)
        error_response["message"] = response.data.get("detail", str(exc))
        error_response["details"] = response.data if isinstance(response.data, dict) else str(response.data)
        
        # Tùy chỉnh cho MethodNotAllowed
        if isinstance(exc, MethodNotAllowed):
            error_response["error_code"] = "invalid_method"
            error_response["message"] = f"Method '{exc}' is not allowed for this endpoint."
        
        response.data = error_response
    else:
        # Để middleware xử lý lỗi non-DRF
        return None

    return response
```

Phản hồi cho `MethodNotAllowed` bây giờ sẽ là:
```json
{
  "status": "error",
  "error_code": "invalid_method",
  "message": "Method 'POST' is not allowed for this endpoint.",
  "details": "Method \"POST\" not allowed.",
  "timestamp": "2025-05-09T12:34:56.789123+00:00"
}
```

---

### 5. Kiểm tra lỗi Invalid HTTP Method và Permission
Để kiểm tra các lỗi này, bạn có thể sử dụng bài kiểm tra tự động hoặc kiểm tra thủ công.

#### Kiểm tra tự động
```python
from django.test import TestCase
from rest_framework.test import APIClient
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
import json

class TestView(APIView):
    def get(self, request):
        return Response({"message": "Success"})

class SecureView(APIView):
    permission_classes = [IsAuthenticated]
    def get(self, request):
        return Response({"message": "Success"})

urlpatterns = [
    path('test/', TestView.as_view()),
    path('secure/', SecureView.as_view()),
]

class TestAPIExceptions(TestCase):
    def setUp(self):
        self.client = APIClient()

    def test_invalid_method(self):
        response = self.client.post('/test/')
        self.assertEqual(response.status_code, 405)
        data = json.loads(response.content)
        self.assertEqual(data['status'], 'error')
        self.assertEqual(data['error_code'], 'invalid_method')
        self.assertEqual(data['message'], "Method 'POST' is not allowed for this endpoint.")
        self.assertTrue('details' in data)
        self.assertTrue('timestamp' in data)

    def test_permission_denied(self):
        response = self.client.get('/secure/')
        self.assertEqual(response.status_code, 403)
        data = json.loads(response.content)
        self.assertEqual(data['status'], 'error')
        self.assertEqual(data['error_code'], 'permission_denied')
        self.assertEqual(data['message'], "Authentication credentials were not provided.")
        self.assertTrue('details' in data)
        self.assertTrue('timestamp' in data)
```

#### Kiểm tra thủ công
1. Chạy server:
   ```bash
   python manage.py runserver
   ```
2. **Invalid HTTP Method**:
   ```bash
   curl -X POST http://127.0.0.1:8000/test/
   ```
   Kiểm tra phản hồi có `error_code: "invalid_method"` và mã trạng thái 405.
3. **Permission Denied**:
   ```bash
   curl -X GET http://127.0.0.1:8000/secure/
   ```
   Kiểm tra phản hồi có `error_code: "permission_denied"` và mã trạng thái 403.

---

### 6. Kết luận
- **Invalid HTTP Method** (`MethodNotAllowed`):
  - **Bắt bởi**: DRF view layer.
  - **Xử lý bởi**: DRF `exception_handler` (tùy chỉnh trong `custom_exception_handler`).
  - **Mã trạng thái**: 405.
- **Permission Denied** (`PermissionDenied`):
  - **Trong DRF**: Bắt bởi DRF permission layer, xử lý bởi `exception_handler` (HTTP 403).
  - **Ngoài DRF**: Bắt bởi middleware (`CustomErrorMiddleware`) (HTTP 403).
- **Cấu trúc lỗi**: Cả hai lỗi được định dạng thống nhất nhờ `custom_exception_handler` (cho DRF) và `CustomErrorMiddleware` (cho non-DRF).

Nếu bạn cần thêm ví dụ (như xử lý lỗi khác hoặc kiểm tra với các trường hợp cụ thể), hãy cho tôi biết!