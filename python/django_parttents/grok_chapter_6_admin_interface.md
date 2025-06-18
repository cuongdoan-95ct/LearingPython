### Chương 6: Giao diện Admin (Tổng quan)

Chương này tập trung vào **giao diện admin** của Django, một công cụ mạnh mẽ tích hợp sẵn giúp quản lý dữ liệu ứng dụng. Nó giải thích cách sử dụng admin hiệu quả, tùy chỉnh giao diện, và bảo vệ nó.

#### Nội dung chính:
1. **Sử dụng giao diện Admin**
2. **Tăng cường Models cho Admin**
3. **Không phải ai cũng nên là Admin**
4. **Tùy chỉnh giao diện Admin**
5. **Bảo vệ Admin**
6. **Mẫu Feature Flags**

---

### 1. Sử dụng giao diện Admin (Using the Admin Interface)

Giao diện admin của Django cho phép quản lý dữ liệu (thêm, sửa, xóa) mà không cần viết code giao diện từ đầu. Nó tự động tạo dựa trên các **models**.

#### Ví dụ cơ bản:
```python
# models.py
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.IntegerField()

    def __str__(self):
        return self.ten
```
```python
# admin.py
from django.contrib import admin
from .models import Product

admin.site.register(Product)
```
- Sau khi đăng ký `Product` trong `admin.py`, bạn có thể truy cập `/admin/` (sau khi đăng nhập với tài khoản superuser) để quản lý sản phẩm.

---

### 2. Tăng cường Models cho Admin (Enhancing Models for the Admin)

Bạn có thể tùy chỉnh cách model hiển thị trong admin bằng cách sử dụng lớp `ModelAdmin`.

#### Ví dụ tùy chỉnh:
```python
# admin.py
from django.contrib import admin
from .models import Product

class ProductAdmin(admin.ModelAdmin):
    list_display = ("ten", "gia")  # Hiển thị cột tên và giá
    list_filter = ("gia",)        # Bộ lọc theo giá
    search_fields = ("ten",)      # Tìm kiếm theo tên

admin.site.register(Product, ProductAdmin)
```
- **`list_display`**: Hiển thị danh sách sản phẩm với cột "tên" và "giá".
- **`list_filter`**: Thêm bộ lọc để lọc sản phẩm theo giá.
- **`search_fields`**: Thêm thanh tìm kiếm theo tên sản phẩm.

#### Kết quả:
- Truy cập `/admin/sanpham/sanpham/` sẽ thấy danh sách sản phẩm với các tính năng lọc và tìm kiếm.

---

### 3. Không phải ai cũng nên là Admin (Not Everyone Should Be an Admin)

Giao diện admin rất mạnh mẽ, nhưng không nên để tất cả người dùng truy cập vì lý do bảo mật. Chỉ **superuser** hoặc người dùng có quyền cụ thể mới nên vào được.

#### Cách giới hạn:
- Sử dụng hệ thống quyền của Django (`permissions`).
- Tạo tài khoản superuser:
```bash
python manage.py createsuperuser
```
- Chỉ cấp quyền admin cho người cần thiết.

---

### 4. Tùy chỉnh giao diện Admin (Admin Interface Customizations)

Bạn có thể thay đổi giao diện admin để phù hợp hơn với nhu cầu.

#### a. Thay đổi tiêu đề (Changing the Heading)
```python
# admin.py
admin.site.site_header = "Quản lý cửa hàng"
admin.site.site_title = "Admin cửa hàng"
```
- Tiêu đề trang admin sẽ hiển thị là "Quản lý cửa hàng".

#### b. Thay đổi base và stylesheets (Changing the Base and Stylesheets)
- Thêm CSS tùy chỉnh:
```python
# admin.py
class ProductSanPhamAdmin(admin.ModelAdmin):
    list_display = ("name", "price")
    
    class Media:
        css = {
            "all": ("css/admin_custom.css",)
        }

admin.site.register(Product, ProductAdmin)
```
```css
/* static/css/admin_custom.css */
body {
    background-color: #f0f0f0;
}
```
- Giao diện admin sẽ có nền màu xám nhạt.

#### c. Thêm trình soạn thảo rich-text (Adding a Rich-Text Editor for WYSIWYG Editing)
- Dùng gói như `django-ckeditor`:
```bash
pip install django-ckeditor
```
```python
# settings.py
INSTALLED_APPS += ["ckeditor"]

# models.py
from ckeditor.fields import RichTextField

class BaiViet(models.Model):
    noi_dung = RichTextField()

# admin.py
admin.site.register(BaiViet)
```
- Trường `noi_dung` sẽ có trình soạn thảo WYSIWYG trong admin.

#### d. Giao diện Bootstrap (Bootstrap-themed Admin)
- Sử dụng gói như `django-bootstrap-admin`:
```bash
pip install django-bootstrap-admin
```
```python
# settings.py
INSTALLED_APPS += ["bootstrap_admin"]
```
- Admin sẽ có giao diện giống Bootstrap.

#### e. Tùy chỉnh hoàn toàn (Complete Overhauls)
- Viết lại template admin:
```python
# settings.py
TEMPLATES[0]["DIRS"] = [BASE_DIR / "templates"]

# templates/admin/base_site.html
{% extends "admin/base.html" %}
{% block title %}{{ title }} | Quản lý mới{% endblock %}
{% block branding %}
    <h1>Admin tùy chỉnh</h1>
{% endblock %}
```
- Giao diện admin sẽ được thay đổi theo template tùy chỉnh.

---

### 5. Bảo vệ Admin (Protecting the Admin)

Admin cần được bảo vệ để tránh truy cập trái phép.

#### Cách bảo vệ:
- **Đặt URL admin khác**: Mặc định là `/admin/`, nên đổi để khó đoán.
```python
# urls.py
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path("quan-ly-he-thong/", admin.site.urls),  # Đổi từ /admin/ thành /quan-ly-he-thong/
]
```
- **Yêu cầu đăng nhập**: Admin luôn yêu cầu đăng nhập, nhưng có thể thêm lớp bảo vệ bổ sung như middleware kiểm tra IP.

---

### 6. Mẫu Feature Flags (Pattern - Feature Flags)

**Feature Flags** là cách bật/tắt tính năng trong admin mà không cần thay đổi code.

#### Vấn đề:
- Cần thử nghiệm hoặc ẩn một số tính năng trong admin.

#### Giải pháp:
- Dùng biến cấu hình hoặc model để bật/tắt tính năng.

#### Ví dụ:
```python
# models.py
class CaiDat(models.Model):
    ten = models.CharField(max_length=100)
    bat = models.BooleanField(default=False)

# admin.py
class SanPhamAdmin(admin.ModelAdmin):
    list_display = ("ten", "gia", "hien_thi_chi_tiet")

    def hien_thi_chi_tiet(self, obj):
        from django.http import HttpResponse
        if CaiDat.objects.filter(ten="hien_chi_tiet", bat=True).exists():
            return "Có"
        return "Không"
    hien_thi_chi_tiet.short_description = "Chi tiết"

admin.site.register(SanPham, SanPhamAdmin)
```
- Nếu trong model `CaiDat` có bản ghi `hien_chi_tiet` với `bat=True`, cột "Chi tiết" sẽ hiển thị "Có", ngược lại là "Không".

#### Ứng dụng Feature Flags:
- **A/B Testing**: Thử nghiệm giao diện khác nhau.
- **Hiệu suất**: Bật/tắt tính năng nặng để kiểm tra hiệu suất.
- **Giới hạn**: Ẩn tính năng với một số người dùng.

---

### Tóm tắt
- **Sử dụng Admin**: Quản lý dữ liệu dễ dàng với giao diện tích hợp.
- **Tăng cường Models**: Tùy chỉnh hiển thị, lọc, tìm kiếm.
- **Tùy chỉnh**: Thay đổi giao diện từ đơn giản (tiêu đề) đến phức tạp (Bootstrap, template mới).
- **Bảo vệ**: Đổi URL, giới hạn truy cập.
- **Feature Flags**: Linh hoạt bật/tắt tính năng.

Hy vọng phần giải thích này giúp bạn hiểu rõ chương 6! Nếu bạn cần thêm ví dụ hoặc chi tiết, hãy cho tôi biết nhé!