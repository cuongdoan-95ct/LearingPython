### Chương 7: Forms

Chương này tập trung vào cách **forms** (biểu mẫu) hoạt động trong Django, từ quy trình xử lý biểu mẫu, hiển thị chúng một cách trực quan, đến các mẫu thiết kế (patterns) giúp tối ưu hóa việc sử dụng biểu mẫu trong ứng dụng web.

#### Nội dung chính:
1. **Forms hoạt động như thế nào**
2. **Forms trong Django**
3. **Hiển thị Forms**
4. **Xử lý Forms với Class-based Views**
5. **Các mẫu Form (Form Patterns)**
---


### 1. Forms hoạt động như thế nào (How Forms Work)
**Forms** trong web là cách người dùng gửi dữ liệu đến server (ví dụ: đăng nhập, đăng ký). Trong Django, quy trình xử lý biểu mẫu bao gồm các bước:

- **Biểu mẫu trống (Empty Form)**: Hiển thị ban đầu cho người dùng điền.
- **Biểu mẫu đã điền (Filled Form)**: Người dùng gửi dữ liệu.
- **Biểu mẫu gửi không lỗi (Submitted Form without Errors)**: Dữ liệu hợp lệ, được xử lý.
- **Biểu mẫu gửi có lỗi (Submitted Form with Errors)**: Dữ liệu không hợp lệ, hiển thị lỗi.

#### Ví dụ cơ bản:
```python
# views.py
from django.shortcuts import render

def dang_ky(request):
    if request.method == "POST":
        ten = request.POST.get("ten")
        email = request.POST.get("email")
        return render(request, "dang_ky_thanh_cong.html", {"ten": ten})
    return render(request, "dang_ky.html")
```
```html
<!-- templates/dang_ky.html -->
<form method="post">
    Tên: <input type="text" name="ten"><br>
    Email: <input type="text" name="email"><br>
    <button type="submit">Đăng ký</button>
</form>
```
- Người dùng nhập tên và email, nhấn "Đăng ký". Nếu là POST, view lấy dữ liệu và render trang thành công.

---

### 2. Forms trong Django (Forms in Django)
Django cung cấp lớp `Form` để đơn giản hóa việc xử lý biểu mẫu, bao gồm xác thực dữ liệu và bảo mật.

#### Tại sao cần làm sạch dữ liệu (Why does data need cleaning)?
- Dữ liệu người dùng gửi có thể không an toàn hoặc không đúng định dạng (ví dụ: email không hợp lệ).
- Django "làm sạch" dữ liệu để đảm bảo nó phù hợp với yêu cầu.

#### Ví dụ Form trong Django:
```python
# forms.py
from django import forms

class DangKyForm(forms.Form):
    ten = forms.CharField(max_length=100, label="Tên")
    email = forms.EmailField(label="Email")
```
```python
# views.py
from django.shortcuts import render
from .forms import DangKyForm

def dang_ky(request):
    if request.method == "POST":
        form = DangKyForm(request.POST)
        if form.is_valid():  # Kiểm tra dữ liệu hợp lệ
            ten = form.cleaned_data["ten"]
            email = form.cleaned_data["email"]
            return render(request, "dang_ky_thanh_cong.html", {"ten": ten})
    else:
        form = DangKyForm()
    return render(request, "dang_ky.html", {"form": form})
```
```html
<!-- templates/dang_ky.html -->
<form method="post">
    {% csrf_token %}
    {{ form.as_p }} <!-- Hiển thị form dưới dạng đoạn văn -->
    <button type="submit">Đăng ký</button>
</form>
```
- `DangKyForm` định nghĩa các trường và kiểm tra dữ liệu.
- `is_valid()` xác thực dữ liệu; nếu hợp lệ, dữ liệu sạch (`cleaned_data`) được sử dụng.

#### CSRF (Cross-Site Request Forgery):
- `{% csrf_token %}` bảo vệ chống tấn công CSRF, đảm bảo yêu cầu đến từ nguồn hợp lệ.

---

### 3. Hiển thị Forms (Displaying Forms)

Django cho phép hiển thị biểu mẫu theo nhiều cách, nhưng có thể làm đẹp hơn với các công cụ như `django-crispy-forms`.

#### Các cách hiển thị cơ bản:
- `{{ form.as_table }}`: Hiển thị dưới dạng bảng.
- `{{ form.as_p }}`: Hiển thị dưới dạng đoạn văn.
- Tùy chỉnh thủ công:
```html
<!-- templates/dang_ky.html -->
<form method="post">
    {% csrf_token %}
    <label>{{ form.ten.label }}</label> {{ form.ten }}<br>
    <label>{{ form.email.label }}</label> {{ form.email }}<br>
    <button type="submit">Đăng ký</button>
</form>
```

#### Dùng django-crispy-forms:
- Cài đặt: `pip install django-crispy-forms`
- Cấu hình `settings.py`:
```python
INSTALLED_APPS += ["crispy_forms"]
CRISPY_TEMPLATE_PACK = "bootstrap4"
```
- Sử dụng:
```python
# forms.py
from crispy_forms.helper import FormHelper
from crispy_forms.layout import Submit

class DangKyForm(forms.Form):
    ten = forms.CharField(max_length=100, label="Tên")
    email = forms.EmailField(label="Email")

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.helper = FormHelper()
        self.helper.add_input(Submit("submit", "Đăng ký"))
```
```html
<!-- templates/dang_ky.html -->
{% load crispy_forms_tags %}
<form method="post">
    {% csrf_token %}
    {% crispy form %}
</form>
```
- Kết quả: Biểu mẫu đẹp mắt với kiểu dáng Bootstrap 4.

---

### 4. Xử lý Forms với Class-based Views (Form Processing with Class-based Views)

**Class-based Views (CBVs)** giúp xử lý biểu mẫu một cách có cấu trúc hơn so với hàm view.

#### Ví dụ CBV:
```python
# views.py
from django.views.generic.edit import FormView
from .forms import DangKyForm

class DangKyView(FormView):
    template_name = "dang_ky.html"
    form_class = DangKyForm
    success_url = "/thanh-cong/"

    def form_valid(self, form):
        ten = form.cleaned_data["ten"]
        return render(self.request, "dang_ky_thanh_cong.html", {"ten": ten})
```
- `FormView` tự động xử lý GET (hiển thị form) và POST (xác thực và xử lý).
- `form_valid` được gọi khi dữ liệu hợp lệ.

#### Mẫu PRG (Post/Redirect/Get):
- Sau khi xử lý POST thành công, CBV chuyển hướng (`success_url`) để tránh gửi lại dữ liệu khi người dùng làm mới trang.

---

### 5. Các mẫu Form (Form Patterns)

Chương này giới thiệu 4 mẫu thiết kế hữu ích cho biểu mẫu:

#### a. Dynamic Form Generation (Tạo biểu mẫu động)
- **Vấn đề**: Tạo biểu mẫu dựa trên dữ liệu động (ví dụ: số trường thay đổi).
- **Giải pháp**: Tạo trường trong `__init__`.

#### Ví dụ:
```python
# forms.py
class SanPhamForm(forms.Form):
    def __init__(self, *args, so_luong=1, **kwargs):
        super().__init__(*args, **kwargs)
        for i in range(so_luong):
            self.fields[f"ten_sp_{i}"] = forms.CharField(label=f"Tên sản phẩm {i+1}")
```
```python
# views.py
def them_san_pham(request):
    form = SanPhamForm(so_luong=3)  # Tạo form với 3 trường
    return render(request, "them_san_pham.html", {"form": form})
```
- Tạo biểu mẫu với 3 trường tên sản phẩm động.

#### b. User-based Forms (Biểu mẫu dựa trên người dùng)
- **Vấn đề**: Hiển thị biểu mẫu khác nhau tùy theo người dùng (ví dụ: admin thấy nhiều trường hơn).
- **Giải pháp**: Điều chỉnh trường dựa trên `request.user`.

#### Ví dụ:
```python
# forms.py
class ThongTinForm(forms.Form):
    ten = forms.CharField(label="Tên")

    def __init__(self, user, *args, **kwargs):
        super().__init__(*args, **kwargs)
        if user.is_staff:  # Chỉ admin thấy trường này
            self.fields["ghi_chu"] = forms.CharField(label="Ghi chú", required=False)
```
```python
# views.py
def thong_tin(request):
    form = ThongTinForm(request.user)
    return render(request, "thong_tin.html", {"form": form})
```

#### c. Multiple Form Actions per View (Xử lý nhiều hành động trong một view)
- **Vấn đề**: Một biểu mẫu cần hỗ trợ nhiều hành động (ví dụ: "Lưu" và "Xóa").
- **Giải pháp**: Dùng nút khác nhau hoặc tách view.

#### Ví dụ với cùng view:
```python
# views.py
def quan_ly_san_pham(request):
    if request.method == "POST":
        form = SanPhamForm(request.POST)
        if form.is_valid():
            if "luu" in request.POST:
                return render(request, "thanh_cong.html", {"msg": "Đã lưu"})
            elif "xoa" in request.POST:
                return render(request, "thanh_cong.html", {"msg": "Đã xóa"})
    else:
        form = SanPhamForm()
    return render(request, "quan_ly.html", {"form": form})
```
```html
<!-- templates/quan_ly.html -->
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit" name="luu">Lưu</button>
    <button type="submit" name="xoa">Xóa</button>
</form>
```
- Hai nút "Lưu" và "Xóa" gửi cùng form nhưng xử lý khác nhau dựa trên `name`.

#### d. CRUD Views (Views CRUD)
- **Vấn đề**: Xử lý tạo, đọc, cập nhật, xóa (CRUD) trong một view.
- **Giải pháp**: Dùng generic views như `CreateView`, `UpdateView`.

#### Ví dụ:
```python
# views.py
from django.views.generic import CreateView, UpdateView
from django.urls import reverse_lazy
from .models import SanPham

class SanPhamCreateView(CreateView):
    model = SanPham
    fields = ["ten", "gia"]
    template_name = "sanpham_form.html"
    success_url = reverse_lazy("danh_sach")

class SanPhamUpdateView(UpdateView):
    model = SanPham
    fields = ["ten", "gia"]
    template_name = "sanpham_form.html"
    success_url = reverse_lazy("danh_sach")
```
```python
# urls.py
from django.urls import path
from .views import SanPhamCreateView, SanPhamUpdateView

urlpatterns = [
    path("them/", SanPhamCreateView.as_view(), name="them"),
    path("sua/<int:pk>/", SanPhamUpdateView.as_view(), name="sua"),
]
```
- `CreateView` để thêm sản phẩm, `UpdateView` để sửa.
---

### Tóm tắt
- **Quy trình**: Biểu mẫu từ trống → điền → xử lý (hợp lệ hoặc lỗi).
- **Django Forms**: Xác thực, làm sạch dữ liệu, bảo vệ CSRF.
- **Hiển thị**: Dùng `crispy-forms` để đẹp hơn.
- **CBVs**: Xử lý biểu mẫu dễ dàng với `FormView`.
- **Patterns**: Tạo động, dựa trên người dùng, nhiều hành động, CRUD.