Có thể áp dụng cách tiếp cận này cho **error response** bằng cách đảm bảo mọi lỗi đều trả về theo format `{ "data": ... }`. Bạn có thể sử dụng **mixin** hoặc tùy chỉnh `exception handler` của Django REST framework.

---

### 🏗 **Cách 1: Sử dụng Mixin cho Error Response**
Nếu muốn đảm bảo tất cả response lỗi tuân theo cấu trúc `{ "data": ... }`, bạn có thể tạo một **ErrorResponseMixin**:

```python
from rest_framework.response import Response
from rest_framework.views import exception_handler

class ErrorResponseMixin:
    def handle_exception(self, exc):
        response = super().handle_exception(exc)
        if response is not None:
            response.data = {"data": response.data}
        return response
```

Sau đó, chỉ cần kế thừa mixin này trong `ViewSet`:

```python
from rest_framework.viewsets import ModelViewSet

class MyViewSet(ModelViewSet, ErrorResponseMixin):
    queryset = MyModel.objects.all()
    serializer_class = MySerializer
```

📌 **Kết quả**:  
✔️ Mọi lỗi đều được trả về theo format `{ "data": ... }`.  
✔️ Không cần chỉnh sửa từng API riêng lẻ.  

---

### 🔧 **Cách 2: Tùy chỉnh Django REST Framework Exception Handler**
Một cách tiếp cận khác là **tùy chỉnh `exception_handler`** để mọi response lỗi luôn theo format `{ "data": ... }`.

📌 **Định nghĩa exception handler trong `utils.py`**:
```python
from rest_framework.views import exception_handler

def custom_exception_handler(exc, context):
    response = exception_handler(exc, context)
    if response is not None:
        response.data = {"data": response.data}
    return response
```

📌 **Tích hợp vào `settings.py`**:
```python
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'path.to.utils.custom_exception_handler',
}
```

🚀 **Tác dụng:**  
✔️ **Tự động áp dụng cho toàn bộ hệ thống**, không cần sửa từng `ViewSet`.  
✔️ **Swagger vẫn hiển thị đúng format**, vì mọi response đều tuân theo `{ "data": ... }`.  
✔️ **Dễ bảo trì**, nếu muốn thay đổi format lỗi, chỉ cần sửa `custom_exception_handler`.  

---

### 🎯 **Kết luận**
✅ **Dùng mixin** nếu chỉ muốn áp dụng cho một số `ViewSet` nhất định.  
✅ **Dùng `exception_handler`** nếu muốn áp dụng **cho toàn bộ API trong hệ thống**.  

Bạn muốn áp dụng cách nào cho hệ thống của bạn? Mình có thể giúp bạn tối ưu hơn nữa! 🚀😃
