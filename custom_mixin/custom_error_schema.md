Có thể áp dụng cách **custom schema** như trên cho **error response**, bằng cách tạo một schema tùy chỉnh cho các mã lỗi phổ biến như **404 (Not Found), 403 (Forbidden), 401 (Unauthorized)**.  

---

### 🔧 **1️⃣ Tạo Serializer chung cho Error Response**
Bạn có thể tạo một serializer chung để định dạng lỗi theo cấu trúc:

```python
from rest_framework import serializers

class ErrorResponseSerializer(serializers.Serializer):
    data = serializers.DictField(child=serializers.JSONField(), default={
        "error_code": "string",
        "message": "string",
        "details": {}
    })
```

🔹 **Cấu trúc của lỗi trên Swagger:**
```json
{
    "data": {
        "error_code": "404",
        "message": "Resource not found",
        "details": {}
    }
}
```

---

### 🏗 **2️⃣ Tùy chỉnh OpenAPI Schema cho Lỗi**
Tương tự như cách tùy chỉnh schema cho response, bạn có thể **định nghĩa `OpenApiSerializerExtension`** để áp dụng tự động cho mọi error response:

```python
from drf_spectacular.extensions import OpenApiSerializerExtension

class CustomErrorSchema(OpenApiSerializerExtension):
    target_class = 'path.to.serializers.ErrorResponseSerializer'

    def get_name(self):
        return "ErrorResponse"

    def get_schema(self, direction):
        schema = super().get_schema(direction)
        return {
            "type": "object",
            "properties": {
                "data": schema
            }
        }
```

📌 **Tác dụng:**  
✔️ **Tự động áp dụng cho mọi error response** mà không cần `@extend_schema`.  
✔️ **Mọi lỗi đều có format chuẩn `{ "data": ... }`** trên Swagger.  

---

### 🔧 **3️⃣ Tích hợp vào `settings.py`**
Sau khi tạo schema tùy chỉnh, bạn cần đăng ký trong `settings.py`:

```python
SPECTACULAR_SETTINGS = {
    'EXTENSIONS': [
        'path.to.schemas.CustomErrorSchema',
    ],
}
```

---

### 🎯 **4️⃣ Kết quả trên Swagger**
✔️ Mọi error response trong hệ thống sẽ có format `{ "data": { "error_code": ..., "message": ... } }`.  
✔️ Không cần chỉnh từng API, schema tự động hiển thị đúng format.  
✔️ API dễ bảo trì và chuẩn hóa hơn.  

---

Nếu bạn muốn áp dụng cho tất cả lỗi của Django REST Framework, bạn cũng có thể **tùy chỉnh `exception_handler`** để tự động dùng schema này cho mọi exception.  

Bạn thử áp dụng nhé! Nếu có vấn đề cần tinh chỉnh, mình sẵn sàng giúp 🚀😃



Chắc chắn rồi! Mình sẽ hướng dẫn bạn cách **tùy chỉnh exception_handler** để đảm bảo mọi lỗi của Django REST Framework luôn tuân theo format `{ "data": ... }`.  

---

### 🔧 **1️⃣ Tạo Exception Handler Tùy Chỉnh**
Bạn có thể viết một `custom_exception_handler` trong `utils.py` để áp dụng cho toàn bộ hệ thống:

```python
from rest_framework.views import exception_handler
from rest_framework.exceptions import APIException

def custom_exception_handler(exc, context):
    response = exception_handler(exc, context)

    # Nếu response tồn tại, áp dụng format { "data": ... }
    if response is not None:
        formatted_response = {
            "data": {
                "error_code": response.status_code,
                "message": response.data.get("detail", "An error occurred"),
                "details": response.data
            }
        }
        response.data = formatted_response

    return response
```

📌 **Điểm chính:**  
✔️ Mọi lỗi (401, 403, 404, 500, ...) sẽ được định dạng thành `{ "data": { error_code, message, details } }`.  
✔️ Thông tin lỗi vẫn đầy đủ, nhưng được hiển thị theo format chuẩn.  

---

### 🔧 **2️⃣ Đăng ký Handler trong `settings.py`**
Sau khi tạo `custom_exception_handler`, bạn cần **đăng ký nó vào Django REST Framework**:

```python
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'path.to.utils.custom_exception_handler',
}
```

📌 **Tác dụng:**  
✔️ **Tự động áp dụng** cho mọi API của hệ thống.  
✔️ **Không cần chỉnh từng ViewSet**, chỉ cần định nghĩa một lần.  
✔️ **Dữ liệu lỗi hiển thị đồng nhất trên Swagger và response thực tế**.  

---

### 🏗 **3️⃣ Demo Response API Lỗi**
Ví dụ, nếu một người dùng gọi API `/notifications/999/` nhưng ID `999` không tồn tại, response sẽ trả về:

```json
{
    "data": {
        "error_code": 404,
        "message": "Not found.",
        "details": {
            "detail": "Not found."
        }
    }
}
```

🚀 **API của bạn sẽ chuyên nghiệp hơn, có cấu trúc rõ ràng và dễ xử lý phía frontend**.  

Bạn thử áp dụng nhé! Nếu cần tinh chỉnh thêm, mình sẵn sàng giúp 😃  
