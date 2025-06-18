### Chương 4: Views và URLs (Tổng quan)

Chương này tập trung vào cách các **views** (chế độ xem) và **URLs** hoạt động trong Django, từ cách các hàm view dựa trên chức năng (function-based views) tiến hóa thành các view dựa trên lớp (class-based views), đến cách thiết kế các URL ngắn gọn và có ý nghĩa. Nó cũng giới thiệu các mẫu (patterns) hữu ích để viết code views hiệu quả hơn.

#### Nội dung chính:
1. **Tổng quan về Views**
2. **Views dựa trên lớp (Class-based Views)**
3. **Mixins trong Views**
4. **Decorators**
5. **Các mẫu View (View Patterns)**
6. **Thiết kế URLs**
7. **Các công cụ thay thế như React.js, Vue.js**

---

### 1. Tổng quan về Views (A View from the Top)

**Views** trong Django là nơi xử lý logic chính của ứng dụng web. Khi một yêu cầu (request) từ trình duyệt được gửi đến, Django sẽ tìm một view tương ứng để xử lý và trả về phản hồi (response), thường là một trang HTML.

- **Trước đây**: Views thường được viết dưới dạng các hàm đơn giản (function-based views).
- **Bây giờ**: Django khuyến khích sử dụng **class-based views** vì chúng dễ mở rộng và tái sử dụng hơn.

#### Ví dụ đơn giản (Function-based View):
```python
from django.http import HttpResponse

def hello(request):
    return HttpResponse("Hello! This is simple view!.")
```
- Khi người dùng truy cập URL liên kết với hàm `hello`, họ sẽ nhận được dòng chữ "Hello! This is simple view!."

---

### 2. Views dựa trên lớp (Class-based Views)

Django giới thiệu **class-based views** (CBVs) để thay thế các hàm view, giúp code có cấu trúc hơn và dễ bảo trì.

#### Ví dụ Class-based View:
```python
from django.views import View
from django.http import HttpResponse

class HelloView(View):
    def get(self, request):
        return HttpResponse("Hello from Class-based View!")
```
- Trong ví dụ này, `HelloView` là một lớp kế thừa từ `View`. Phương thức `get` xử lý yêu cầu GET.

#### Class-based Generic Views:
Django cung cấp các generic views (views chung) như `ListView`, `DetailView` để giảm bớt công việc lặp lại.

#### Ví dụ Generic View:
```python
from django.views.generic import ListView
from .models import SanPham

class DanhSachSanPhamView(ListView):
    model = SanPham
    template_name = "sanpham/danh_sach.html"
```
- `DanhSachSanPhamView` hiển thị danh sách tất cả các sản phẩm từ model `SanPham` và render bằng template `danh_sach.html`.

---

### 3. Mixins trong Views (View Mixins)

**Mixins** là các lớp nhỏ bổ sung chức năng cho class-based views. Chúng giúp tái sử dụng code và tránh lặp lại.

#### Ví dụ Mixin:
```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import ListView
from .models import SanPham

class DanhSachSanPhamDaDangNhapView(LoginRequiredMixin, ListView):
    model = SanPham
    template_name = "sanpham/danh_sach.html"
```
- `LoginRequiredMixin` yêu cầu người dùng phải đăng nhập để truy cập view. Nếu chưa đăng nhập, họ sẽ được chuyển hướng đến trang đăng nhập.

**Thứ tự Mixins**: Mixins phải được đặt trước lớp generic view trong danh sách kế thừa (vì Python xử lý theo **Method Resolution Order - MRO**).

---

### 4. Decorators

**Decorators** là cách áp dụng logic bổ sung (như kiểm tra quyền truy cập) cho views. Với CBVs, bạn có thể áp dụng decorator lên phương thức cụ thể.

#### Ví dụ Decorator:
```python
from django.contrib.auth.decorators import login_required
from django.utils.decorators import method_decorator
from django.views import View
from django.http import HttpResponse

class TrangBiMatView(View):
    @method_decorator(login_required)
    def get(self, request):
        return HttpResponse("Chỉ người dùng đăng nhập mới thấy được!")
```
- `@method_decorator(login_required)` đảm bảo chỉ người dùng đã đăng nhập mới truy cập được view này.

- Decorators ít linh hoạt hơn mixin.
- Tuy nhiên, chúng đơn giản hơn. Bạn có thể sử dụng cả hai
- Trên thực tế, nhiều mixin được triển khai bằng decorator.

---

### 5. Các mẫu View (View Patterns)

Chương này giới thiệu 3 mẫu view hữu ích:

#### a. Access Controlled Views (Views kiểm soát truy cập)
- **Vấn đề**: Giới hạn quyền truy cập vào một số trang nhất định (ví dụ: chỉ admin mới xem được).
- **Giải pháp**: Sử dụng decorator hoặc mixin để kiểm tra quyền.

#### Ví dụ:
```python
from django.contrib.auth.mixins import PermissionRequiredMixin
from django.views.generic import ListView
from .models import SanPham

class DanhSachSanPhamAdminView(PermissionRequiredMixin, ListView):
    model = SanPham
    template_name = "sanpham/danh_sach_admin.html"
    permission_required = "sanpham.view_sanpham"
```
- Chỉ người dùng có quyền `sanpham.view_sanpham` mới truy cập được.

#### b. Context Enhancers (Tăng cường ngữ cảnh)
- **Vấn đề**: Thêm dữ liệu bổ sung vào context của template mà không làm lộn xộn logic view.
- **Giải pháp**: Sử dụng phương thức `get_context_data`.

#### Ví dụ:
```python
from django.views.generic import DetailView
from .models import SanPham

class ChiTietSanPhamView(DetailView):
    model = SanPham
    template_name = "sanpham/chi_tiet.html"

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["san_pham_lien_quan"] = SanPham.objects.filter(danh_muc=self.object.danh_muc)[:3]
        return context
```
- View này hiển thị chi tiết sản phẩm và thêm 3 sản phẩm liên quan vào context.

#### c. Services (Dịch vụ)
- **Vấn đề**: Tách logic phức tạp ra khỏi view để dễ bảo trì.
- **Giải pháp**: Tạo các hàm dịch vụ riêng.

#### Ví dụ:
```python
from django.http import JsonResponse
from django.views import View

def tinh_tong_gio_hang(gio_hang):
    return sum(item["gia"] * item["so_luong"] for item in gio_hang)

class TongGioHangView(View):
    def get(self, request):
        gio_hang = [{"gia": 100, "so_luong": 2}, {"gia": 50, "so_luong": 1}]
        tong = tinh_tong_gio_hang(gio_hang)
        return JsonResponse({"tong": tong})
```
- Hàm `tinh_tong_gio_hang` được tách ra để tính tổng giỏ hàng, giữ view gọn gàng.

---

### 6. Thiết kế URLs (Designing URLs)

URLs trong Django được định nghĩa trong file `urls.py`. Chương này giải thích cách tạo URL ngắn gọn, dễ hiểu và linh hoạt.

#### Cấu trúc URL:
- **Path**: Đường dẫn (ví dụ: `/san-pham/`).
- **Name**: Tên để tham chiếu URL trong code (ví dụ: `"danh-sach-san-pham"`).
- **Namespace**: Nhóm các URL để tránh xung đột.

#### Ví dụ urls.py:
```python
from django.urls import path
from .views import DanhSachSanPhamView, ChiTietSanPhamView

app_name = "sanpham"  # Namespace
urlpatterns = [
    path("danh-sach/", DanhSachSanPhamView.as_view(), name="danh-sach"),
    path("chi-tiet/<int:pk>/", ChiTietSanPhamView.as_view(), name="chi-tiet"),
]
```
- `/san-pham/danh-sach/` dẫn đến danh sách sản phẩm.
- `/san-pham/chi-tiet/1/` hiển thị chi tiết sản phẩm có ID là 1.

#### Các kiểu URL:
- **Department Store URLs**: Nhiều cấp như `/san-pham/dien-thoai/iphone/`.
- **RESTful URLs**: Ngắn gọn, mô tả tài nguyên như `/san-pham/1/`.

---

### 7. Các công cụ thay thế (React.js, Vue.js)

Ngoài cách truyền thống (views render templates), bạn có thể dùng Django làm API backend và React/Vue.js làm frontend (Single Page Application - SPA).

#### Ví dụ:
- Django REST API trả về JSON:
```python
from rest_framework.views import APIView
from rest_framework.response import Response

class SanPhamAPI(APIView):
    def get(self, request):
        san_pham = [{"id": 1, "ten": "iPhone"}]
        return Response(san_pham)
```
- URL: `/api/san-pham/`.
- React/Vue.js sẽ lấy dữ liệu JSON này để hiển thị.

---

### Tóm tắt
- **Views**: Chuyển từ function-based sang class-based để dễ mở rộng.
- **Mixins/Decorators**: Giúp kiểm soát truy cập và tái sử dụng code.
- **Patterns**: Tách logic, tăng cường context, kiểm soát truy cập.
- **URLs**: Thiết kế ngắn gọn, dễ hiểu với tên và namespace.
- **SPA**: Kết hợp Django với React/Vue.js cho ứng dụng hiện đại.

Hy vọng phần giải thích này giúp bạn hiểu rõ nội dung chương 4! Nếu bạn cần thêm ví dụ hoặc giải thích chi tiết, hãy cho tôi biết nhé!