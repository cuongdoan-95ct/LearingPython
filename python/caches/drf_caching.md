## **Tổng quan về Caching trong DRF**

**Caching** giúp lưu trữ tạm thời kết quả của các yêu cầu (requests) để giảm tải cho server và tăng tốc độ phản hồi.
DRF tận dụng các tiện ích cache của Django, cho phép bạn dễ dàng áp dụng cache vào các view của mình.

---

## **Sử dụng Cache với `APIView` và `ViewSet`**

Trong DRF, bạn có thể sử dụng các decorator của Django như `cache_page`, `vary_on_cookie`, và `vary_on_headers` để áp dụng cache cho các class-based views như `APIView` và `ViewSet`.

### Ví dụ:

```python
from django.utils.decorators import method_decorator
from django.views.decorators.cache import cache_page, vary_on_cookie, vary_on_headers
from rest_framework.response import Response
from rest_framework.views import APIView
from rest_framework import viewsets

class UserViewSet(viewsets.ViewSet):
    # Cache mỗi URL được yêu cầu cho từng người dùng trong 2 giờ
    @method_decorator(cache_page(60 * 60 * 2))
    @method_decorator(vary_on_cookie)
    def list(self, request, format=None):
        content = {
            "user_feed": request.user.get_user_feed(),
        }
        return Response(content)

class ProfileView(APIView):
    # Cache mỗi URL được yêu cầu cho từng người dùng dựa trên header Authorization trong 2 giờ
    @method_decorator(cache_page(60 * 60 * 2))
    @method_decorator(vary_on_headers("Authorization"))
    def get(self, request, format=None):
        content = {
            "user_feed": request.user.get_user_feed(),
        }
        return Response(content)

class PostView(APIView):
    # Cache trang cho URL được yêu cầu trong 2 giờ
    @method_decorator(cache_page(60 * 60 * 2))
    def get(self, request, format=None):
        content = {
            "title": "Post title",
            "body": "Post content",
        }
        return Response(content)

Trong ví dụ trên:

- **`UserViewSet`**: Cache phản hồi dựa trên cookie của người dùng, đảm bảo mỗi người dùng nhận được dữ liệu riêng của họ.
- **`ProfileView`**: Cache phản hồi dựa trên header `Authorization`, hữu ích khi sử dụng token-based authentication.
- **`PostView`**: Cache phản hồi chung cho tất cả người dùng, không phân biệt.

---

## **Sử dụng Cache với `@api_view` Decorator**

Khi sử dụng decorator `@api_view`, bạn có thể trực tiếp áp dụng các decorator cache của Django.

### Ví dụ:

```python
from django.views.decorators.cache import cache_page, vary_on_cookie
from rest_framework.decorators import api_view
from rest_framework.response import Response

@cache_page(60 * 15)
@vary_on_cookie
@api_view(["GET"])
def get_user_list(request):
    content = {"user_feed": request.user.get_user_feed()}
    return Response(content)
```

Trong ví dụ này, phản hồi của view `get_user_list` sẽ được cache trong 15 phút và phân biệt theo cookie của người dùng.

## **Lưu ý Quan Trọng**

- **Thời gian Cache**: Đặt thời gian cache phù hợp với tính chất động của dữ liệu. Dữ liệu thay đổi thường xuyên nên có thời gian cache ngắn hơn.
- **Phân biệt Người Dùng**: Sử dụng `vary_on_cookie` hoặc `vary_on_headers` để đảm bảo mỗi người dùng nhận được dữ liệu phù hợp với họ.
- **Xóa Cache Khi Dữ Liệu Thay Đổi**: Khi dữ liệu được cập nhật, hãy đảm bảo xóa cache liên quan để tránh cung cấp dữ liệu cũ.

## **Tham khảo thêm**

Để hiểu rõ hơn và cập nhật thông tin mới nhất, bạn có thể tham khảo tài liệu chính thức của Django REST Framework về caching tại: [Caching - Django REST Framework](https://www.django-rest-framework.org/api-guide/caching/)
