### Chương 5: Templates (Tổng quan)
Chương này tập trung vào cách sử dụng **templates** (mẫu) trong Django để tạo giao diện người dùng (UI) một cách linh hoạt và dễ bảo trì. Nó giới thiệu ngôn ngữ mẫu Django (Django Template Language - DTL), cách tổ chức file template, và các mẫu thiết kế (patterns) hữu ích.

#### Nội dung chính:
1. **Ngôn ngữ mẫu của Django (Django Template Language - DTL)**
2. **Tổ chức Templates**
3. **Cách Templates hoạt động**
4. **Sử dụng Bootstrap**
5. **Các mẫu Template (Template Patterns)**
---

### 1. Ngôn ngữ mẫu của Django (Understanding Django's Template Language Features)
**Django Template Language (DTL)** là công cụ chính để tạo giao diện động trong Django. Nó đơn giản, an toàn và không phải là một ngôn ngữ lập trình đầy đủ (theo triết lý của Django).

#### Các thành phần chính:
- **Variables (Biến)**: Hiển thị dữ liệu từ context.
- **Attributes (Thuộc tính)**: Truy cập thuộc tính của đối tượng.
- **Filters (Bộ lọc)**: Thay đổi cách hiển thị dữ liệu.
- **Tags (Thẻ)**: Thực hiện logic như vòng lặp hoặc điều kiện.

#### Ví dụ cơ bản:
```html
<!-- templates/xin_chao.html -->
<h1>Xin chào {{ ten_nguoi_dung|upper }}</h1>
<p>Tuổi: {{ tuoi|default:"Chưa biết" }}</p>
{% if tuoi > 18 %}
    <p>Bạn đã trưởng thành!</p>
{% else %}
    <p>Bạn còn nhỏ!</p>
{% endif %}
```
- `{{ ten_nguoi_dung|upper }}`: Hiển thị tên người dùng in hoa (filter `upper`).
- `{{ tuoi|default:"Chưa biết" }}`: Nếu `tuoi` không có, hiển thị "Chưa biết".
- `{% if %}`: Thẻ điều kiện kiểm tra tuổi.

#### Triết lý: "Đừng tạo ra một ngôn ngữ lập trình"
- DTL được thiết kế đơn giản, tránh logic phức tạp để giữ sự tách biệt giữa logic (views) và giao diện (templates).
---

### 2. Tổ chức Templates (Organizing Templates)

Django không áp đặt cấu trúc thư mục cụ thể cho templates, nhưng có cách tổ chức phổ biến để dễ quản lý.

#### Cách tổ chức đề xuất:
```
project/
    app_name/
        templates/
            app_name/  # Namespace để tránh xung đột
                base.html
                danh_sach.html
                chi_tiet.html
```
- **Namespace**: Đặt template trong thư mục có tên trùng với app (`app_name/`) để tránh trùng lặp tên với các app khác.

#### Ví dụ cấu hình settings.py:
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,  # Tự động tìm templates trong thư mục app
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.request',
            ],
        },
    },
]
```
---

### 3. Cách Templates hoạt động (How Templates Work)

Templates nhận dữ liệu từ views thông qua **context** và render thành HTML để gửi đến trình duyệt.

#### Quy trình:
1. View gửi context (dữ liệu) đến template.
2. Template xử lý dữ liệu với biến, filter, tag.
3. Django render template thành HTML.

#### Ví dụ:
```python
# views.py
from django.shortcuts import render

def danh_sach_san_pham(request):
    san_pham = [{"ten": "iPhone", "gia": 1000}, {"ten": "Samsung", "gia": 800}]
    return render(request, "sanpham/danh_sach.html", {"danh_sach": san_pham})
```
```html
<!-- templates/sanpham/danh_sach.html -->
<ul>
{% for sp in danh_sach %}
    <li>{{ sp.ten }} - {{ sp.gia }} USD</li>
{% endfor %}
</ul>
```
- Kết quả: Một danh sách HTML hiển thị "iPhone - 1000 USD" và "Samsung - 800 USD".
---

### 4. Sử dụng Bootstrap (Using Bootstrap)

**Bootstrap** là một framework CSS giúp giao diện đẹp và responsive. Có thể tích hợp vào Django theo nhiều cách.

#### Cách tích hợp:
- **Tải thủ công**: Copy file CSS/JS từ trang Bootstrap (https://getbootstrap.com).
- **Dùng gói**: Cài qua pip như `django-bootstrap4`.

#### Ví dụ với Bootstrap:
```html
<!-- templates/sanpham/danh_sach.html -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
<div class="container">
    <h1 class="mt-3">Danh sách sản phẩm</h1>
    <ul class="list-group">
    {% for sp in danh_sach %}
        <li class="list-group-item">{{ sp.ten }} - {{ sp.gia }} USD</li>
    {% endfor %}
    </ul>
</div>
```
- Kết quả: Danh sách sản phẩm được định dạng đẹp mắt với Bootstrap.

#### Nhược điểm:
- Giao diện có thể trông giống nhau nếu không tùy chỉnh.

#### Giải pháp thay thế nhẹ:
- Dùng framework nhẹ như Bulma hoặc Tailwind CSS.
---

### 5. Các mẫu Template (Template Patterns)

Chương này giới thiệu 2 mẫu template hữu ích:

#### a. Template Inheritance Tree (Cây kế thừa template)
- **Vấn đề**: Nhiều template lặp lại code (header, footer).
- **Giải pháp**: Sử dụng kế thừa với `{% extends %}` và `{% block %}`.

#### Ví dụ:
```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
</head>
<body>
    <header>Chào mừng đến với trang web</header>
    {% block content %}{% endblock %}
    <footer>© 2023</footer>
</body>
</html>
```
```html
<!-- templates/sanpham/danh_sach.html -->
{% extends "base.html" %}
{% block title %}Danh sách sản phẩm{% endblock %}
{% block content %}
    <ul>
    {% for sp in danh_sach %}
        <li>{{ sp.ten }} - {{ sp.gia }} USD</li>
    {% endfor %}
    </ul>
{% endblock %}
```
- `base.html` là template cơ sở, `danh_sach.html` kế thừa và chỉ thay đổi `title` và `content`.

#### b. The Active Link (Liên kết hoạt động)
- **Vấn đề**: Đánh dấu liên kết đang hoạt động trong menu điều hướng.
- **Giải pháp**: Dùng điều kiện hoặc tag tùy chỉnh.

#### Ví dụ giải pháp đơn giản:
```html
<!-- templates/base.html -->
<nav>
    <a href="{% url 'sanpham:danh-sach' %}" {% if request.path == '/san-pham/danh-sach/' %}class="active"{% endif %}>Danh sách</a>
    <a href="{% url 'sanpham:chi-tiet' 1 %}" {% if request.path == '/san-pham/chi-tiet/1/' %}class="active"{% endif %}>Chi tiết</a>
</nav>
{% block content %}{% endblock %}
```
- `request.path` kiểm tra URL hiện tại để thêm class `"active"`.

#### Giải pháp với Custom Tag:
```python
# templatetags/nav_tags.py
from django import template

register = template.Library()

@register.simple_tag
def active_link(request, url_name):
    from django.urls import reverse
    return "active" if request.path == reverse(url_name) else ""
```
```html
<!-- templates/base.html -->
{% load nav_tags %}
<nav>
    <a href="{% url 'sanpham:danh-sach' %}" class="{% active_link request 'sanpham:danh-sach' %}">Danh sách</a>
    <a href="{% url 'sanpham:chi-tiet' 1 %}" class="{% active_link request 'sanpham:chi-tiet' %}">Chi tiết</a>
</nav>
```
- Tag `active_link` tự động thêm class `"active"` dựa trên URL.
---

### Tóm tắt
- **DTL**: Công cụ đơn giản với biến, filter, tag để hiển thị dữ liệu động.
- **Tổ chức**: Dùng namespace và cấu trúc thư mục rõ ràng.
- **Bootstrap**: Tích hợp để giao diện đẹp, nhưng cần tùy chỉnh để tránh đơn điệu.
- **Patterns**: Kế thừa template giảm lặp code; active link cải thiện điều hướng.