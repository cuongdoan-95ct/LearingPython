Để tùy chỉnh middleware xử lý lỗi trong Django 5.1, bạn có thể tạo một middleware tùy chỉnh để định dạng lại các phản hồi lỗi theo cách bạn muốn. Dưới đây là một ví dụ về cách triển khai middleware để tùy chỉnh phản hồi lỗi, chẳng hạn như trả về JSON với cấu trúc thống nhất cho API.

```python
class CustomErrorMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        response = self.get_response(request)
        return response

    def process_exception(self, request, exception):
        # Xử lý các ngoại lệ và trả về phản hồi lỗi tùy chỉnh
        from django.http import JsonResponse
        import logging

        # Ghi log lỗi nếu cần
        logger = logging.getLogger(__name__)
        logger.error(f"Error occurred: {str(exception)}", exc_info=True)

        # Định dạng phản hồi lỗi
        error_response = {
            "status": "error",
            "message": str(exception),
            "code": getattr(exception, "status_code", 500),
        }

        # Xử lý các loại ngoại lệ cụ thể nếu cần
        if isinstance(exception, ValueError):
            error_response["message"] = "Invalid input provided."
            error_response["code"] = 400
        elif isinstance(exception, PermissionError):
            error_response["message"] = "Permission denied."
            error_response["code"] = 403
        # Thêm các loại ngoại lệ khác nếu cần

        return JsonResponse(error_response, status=error_response["code"])
```

### Hướng dẫn sử dụng:

1. **Lưu middleware**: Lưu mã trên vào một tệp, ví dụ `custom_error_middleware.py`, trong thư mục ứng dụng hoặc thư mục `middleware` của dự án.

2. **Đăng ký middleware**: Thêm middleware vào cài đặt `MIDDLEWARE` trong tệp `settings.py`:

   ```python
   MIDDLEWARE = [
       ...
       'path.to.custom_error_middleware.CustomErrorMiddleware',
   ]
   ```

3. **Giải thích mã**:
   - `__init__`: Khởi tạo middleware với `get_response`.
   - `__call__`: Xử lý yêu cầu và phản hồi thông thường.
   - `process_exception`: Xử lý các ngoại lệ, trả về `JsonResponse` với cấu trúc lỗi tùy chỉnh.
   - Cấu trúc phản hồi lỗi bao gồm `status`, `message`, và `code`.
   - Bạn có thể tùy chỉnh thêm logic trong `process_exception` để xử lý các loại ngoại lệ cụ thể.

4. **Ghi log**: Middleware sử dụng `logging` để ghi lại lỗi, giúp dễ dàng debug.

5. **Tùy chỉnh thêm**:
   - Thêm xử lý cho các ngoại lệ cụ thể (như `Http404`, `ValidationError`, v.v.).
   - Thay đổi cấu trúc JSON theo nhu cầu API của bạn.
   - Thêm các trường như `details` hoặc `error_type` nếu cần.

Middleware này hoạt động tốt với API, đảm bảo mọi lỗi được trả về với định dạng thống nhất. Nếu bạn cần thêm tính năng cụ thể hoặc xử lý lỗi khác, hãy cho tôi biết!