Below is a summary of **Chapter 9: Creating APIs** from *Django Design Patterns and Best Practices, Second Edition* by Arun Ravindran, translated into Vietnamese with explanations and examples to help you understand the content effectively.

---

### Chương 9: Tạo API (Tổng quan)

Chương này tập trung vào việc thiết kế và xây dựng **API RESTful** trong Django, sử dụng **Django REST Framework (DRF)**. Nó giải thích các khái niệm cơ bản, cách thiết kế API, và các mẫu thiết kế (patterns) hữu ích để tối ưu hóa API.

#### Nội dung chính:
1. **API RESTful**
2. **Thiết kế API**
3. **Phiên bản hóa (Versioning)**
4. **Django REST Framework**
5. **Các mẫu API (API Patterns)**

---

### 1. API RESTful (RESTful API)

**REST (Representational State Transfer)** là một phong cách kiến trúc để xây dựng API dựa trên giao thức HTTP. Nó tập trung vào tài nguyên (resources) và sử dụng các phương thức HTTP (GET, POST, PUT, DELETE) để thao tác.

#### Đặc điểm của hệ thống RESTful:
- **Client-Server**: Tách biệt client và server.
- **Stateless**: Mỗi yêu cầu độc lập, không lưu trạng thái.
- **Cacheable**: Có thể lưu trữ dữ liệu để tăng tốc.
- **Layered System**: Hỗ trợ nhiều lớp (ví dụ: proxy).
- **Uniform Interface**: Giao diện nhất quán với tài nguyên.
- **Code on Demand** (tùy chọn): Trả về mã thực thi (như JavaScript).

#### Các yếu tố kiến trúc REST:
- **Resources**: Được biểu diễn qua URI (ví dụ: `/san-pham/`).
- **Request Operations**: GET (lấy), POST (tạo), PUT (cập nhật), DELETE (xóa).
- **Error Codes**: HTTP status (200 OK, 404 Not Found, 400 Bad Request).
- **Hypermedia**: Liên kết đến các tài nguyên liên quan (HATEOAS).

---

### 2. Thiết kế API (API Design)

Thiết kế API cần rõ ràng, dễ hiểu, và tuân theo các tiêu chuẩn REST.

#### Nguyên tắc thiết kế:
- **Định danh tài nguyên**: Dùng danh từ (ví dụ: `/san-pham/` thay vì `/lay-san-pham/`).
- **Phương thức HTTP**: Ánh xạ đúng hành động (GET để lấy, POST để tạo).
- **Trạng thái rõ ràng**: Trả về mã trạng thái phù hợp (201 Created, 400 Bad Request).
- **Hỗ trợ hypermedia**: Thêm liên kết để điều hướng (ví dụ: `"next": "/san-pham/?page=2"`).

#### Ví dụ cơ bản:
- Yêu cầu: `GET /san-pham/`
- Phản hồi:
```json
[
    {"id": 1, "ten": "iPhone", "gia": 1000},
    {"id": 2, "ten": "Samsung", "gia": 800}
]
```

---

### 3. Phiên bản hóa (Versioning)

API cần phiên bản để hỗ trợ thay đổi mà không làm hỏng client cũ.

#### Các cách phiên bản hóa:
- **URI Versioning**: Thêm phiên bản vào URL (ví dụ: `/v1/san-pham/`).
- **Query String Versioning**: Dùng tham số (ví dụ: `/san-pham/?version=1`).
- **Custom Header Versioning**: Dùng header (ví dụ: `Accept: application/vnd.api.v1+json`).
- **Media Type Versioning**: Dùng kiểu nội dung (ví dụ: `Accept: application/vnd.api+json; version=1.0`).

#### Ví dụ URI Versioning:
```python
# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path("v1/san-pham/", views.SanPhamList.as_view()),
    path("v2/san-pham/", views.SanPhamListV2.as_view()),
]
```

---

### 4. Django REST Framework (Django Rest Framework)

**DRF** là thư viện mạnh mẽ để xây dựng API RESTful trong Django.

#### Cài đặt:
```bash
pip install djangorestframework
```
```python
# settings.py
INSTALLED_APPS += ["rest_framework"]
```

#### Ví dụ API cơ bản:
```python
# models.py
from django.db import models

class SanPham(models.Model):
    ten = models.CharField(max_length=100)
    gia = models.IntegerField()

    def __str__(self):
        return self.ten
```
```python
# serializers.py
from rest_framework import serializers
from .models import SanPham

class SanPhamSerializer(serializers.ModelSerializer):
    class Meta:
        model = SanPham
        fields = ["id", "ten", "gia"]
```
```python
# views.py
from rest_framework import generics
from .models import SanPham
from .serializers import SanPhamSerializer

class SanPhamList(generics.ListCreateAPIView):
    queryset = SanPham.objects.all()
    serializer_class = SanPhamSerializer
```
```python
# urls.py
from django.urls import path
from .views import SanPhamList

urlpatterns = [
    path("san-pham/", SanPhamList.as_view(), name="san-pham-list"),
]
```
- Truy cập `/san-pham/` trả về danh sách sản phẩm dưới dạng JSON.

#### Cải thiện API:
- **Ẩn ID**: Dùng slug thay vì ID số.
```python
# serializers.py
class SanPhamSerializer(serializers.ModelSerializer):
    slug = serializers.SlugField(source="ten", read_only=True)
    class Meta:
        model = SanPham
        fields = ["slug", "ten", "gia"]  # Ẩn id
```
- Kết quả:
```json
[
    {"slug": "iphone", "ten": "iPhone", "gia": 1000},
    {"slug": "samsung", "ten": "Samsung", "gia": 800}
]
```

---

### 5. Các mẫu API (API Patterns)

Chương này giới thiệu 2 mẫu thiết kế hữu ích:

#### a. Human Browsable Interface (Giao diện có thể duyệt bằng người)
- **Vấn đề**: API chỉ trả JSON khó đọc trực tiếp trên trình duyệt.
- **Giải pháp**: Thêm giao diện HTML cho người dùng.

##### Ví dụ:
```python
# views.py
from rest_framework.renderers import TemplateHTMLRenderer, JSONRenderer
from rest_framework.views import APIView
from rest_framework.response import Response
from .models import SanPham
from .serializers import SanPhamSerializer

class SanPhamList(APIView):
    renderer_classes = [TemplateHTMLRenderer, JSONRenderer]

    def get(self, request, *args, **kwargs):
        san_pham = SanPham.objects.all()
        serializer = SanPhamSerializer(san_pham, many=True)
        if request.accepted_renderer.format == "html":
            return Response({"san_pham": serializer.data}, template_name="san_pham_list.html")
        return Response(serializer.data)
```
```html
<!-- templates/san_pham_list.html -->
<ul>
{% for sp in san_pham %}
    <li>{{ sp.ten }} - {{ sp.gia }} USD</li>
{% endfor %}
</ul>
```
- Truy cập `/san-pham/`:
  - Trình duyệt: Danh sách HTML.
  - API client: JSON.

#### b. Infinite Scrolling (Cuộn vô hạn)
- **Vấn đề**: Tải toàn bộ dữ liệu gây chậm khi danh sách dài.
- **Giải pháp**: Phân trang và tải dần khi cuộn.

##### Ví dụ:
```python
# views.py
from rest_framework import generics
from rest_framework.pagination import PageNumberPagination
from .models import SanPham
from .serializers import SanPhamSerializer

class SanPhamPagination(PageNumberPagination):
    page_size = 10

class SanPhamList(generics.ListAPIView):
    queryset = SanPham.objects.all()
    serializer_class = SanPhamSerializer
    pagination_class = SanPhamPagination
```
```javascript
// frontend.js
let page = 1;
function loadMore() {
    fetch(`/san-pham/?page=${page}`)
        .then(response => response.json())
        .then(data => {
            data.results.forEach(sp => {
                document.getElementById("list").innerHTML += `<li>${sp.ten} - ${sp.gia}</li>`;
            });
            if (data.next) page++;
            else document.getElementById("loadMore").style.display = "none";
        });
}

window.onscroll = () => {
    if (window.innerHeight + window.scrollY >= document.body.offsetHeight) {
        loadMore();
    }
};
loadMore(); // Tải trang đầu tiên
```
```html
<!-- templates/index.html -->
<ul id="list"></ul>
<button id="loadMore" onclick="loadMore()">Tải thêm</button>
<script src="frontend.js"></script>
```
- API trả về dữ liệu phân trang (10 sản phẩm/lần), frontend tải thêm khi cuộn.

---

### Tóm tắt
- **RESTful**: API dựa trên tài nguyên, HTTP, stateless.
- **Thiết kế**: Dùng danh từ, mã trạng thái, hypermedia.
- **Versioning**: URI, header, query string.
- **DRF**: Xây dựng API dễ dàng, hỗ trợ serializer, view generic.
- **Patterns**: Giao diện người dùng, cuộn vô hạn.