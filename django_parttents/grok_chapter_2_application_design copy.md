## Chương 2: Thiết kế Ứng dụng (Application Design)

Chương này tập trung vào giai đoạn đầu của vòng đời phát triển ứng dụng, từ việc thu thập yêu cầu, thiết kế ban đầu, đến cách tổ chức dự án Django một cách hiệu quả.
Nó cũng giới thiệu dự án mẫu "SuperBook" - một mạng xã hội cho siêu anh hùng - để minh họa các khái niệm.

### 1. Làm thế nào để thu thập yêu cầu? (How to gather requirements?)
#### Giải thích:
- Trước khi bắt đầu viết code, bạn cần hiểu rõ ứng dụng sẽ làm gì. Điều này bắt đầu bằng việc thu thập yêu cầu từ khách hàng hoặc người dùng.
- Một cách hiệu quả là đặt câu hỏi cụ thể: "Ứng dụng cần giải quyết vấn đề gì?" hoặc "Người dùng mong đợi điều gì?"
- Trong thực tế, yêu cầu thường không rõ ràng ngay từ đầu, nên bạn cần liên tục trao đổi để làm rõ.

#### Ví dụ:
- Giả sử bạn xây dựng "SuperBook" (mạng xã hội cho siêu anh hùng):
  - Yêu cầu ban đầu: "Siêu anh hùng có thể đăng bài và xem bài của người khác."
  - Sau khi hỏi thêm: "Cần thêm tính năng bình luận, thích bài viết, và phân quyền (chỉ siêu anh hùng đã đăng ký mới đăng bài được)."

### 2. Bạn có phải là người kể chuyện không? (Are you a storyteller?)
#### Giải thích:
- Việc kể chuyện giúp biến yêu cầu thành một kịch bản cụ thể, dễ hình dung hơn cho cả lập trình viên và khách hàng.
- Sử dụng câu chuyện để mô tả cách người dùng tương tác với ứng dụng.

#### Ví dụ:
- Câu chuyện cho SuperBook:  
  *"Captain Obvious muốn chia sẻ chiến công mới nhất của mình. Anh ấy đăng nhập vào SuperBook,
  viết một bài về việc cứu thế giới khỏi robot khổng lồ, thêm ảnh minh họa, và đăng bài. Hexa nhìn thấy bài viết, thích nó, và để lại bình luận: 'Tuyệt vời!'."*

### 3. HTML Mockups (Mô hình HTML)
#### Giải thích:
- Trước khi code thật, tạo giao diện HTML đơn giản (mockup) giúp hình dung ứng dụng sẽ trông như thế nào.
- Mockup không cần đẹp, chỉ cần thể hiện bố cục và chức năng chính.

#### Ví dụ:
- Mockup cho SuperBook:
```html
<!-- templates/home.html -->
<!DOCTYPE html>
<html>
<head>
    <title>SuperBook</title>
</head>
<body>
    <h1>Chào mừng đến SuperBook</h1>
    <div>
        <h2>Bài viết mới nhất</h2>
        <p><strong>Captain Obvious:</strong> Đánh bại robot khổng lồ!</p>
        <button>Thích</button> <a href="#">Bình luận</a>
    </div>
    <form>
        <textarea placeholder="Viết bài mới..."></textarea>
        <button>Đăng</button>
    </form>
</body>
</html>
```
- Đây là bản nháp giao diện, sau này sẽ được cải thiện bằng Django template.

### 4. Thiết kế ứng dụng (Designing the application)
#### Giải thích:
- Sau khi có yêu cầu và mockup, bạn cần chia ứng dụng thành các phần nhỏ hơn để dễ quản lý.
- Trong Django, điều này liên quan đến việc chia dự án thành các **app**.

### 5. Chia dự án thành các app (Dividing a project into apps)
#### Giải thích:
- Một dự án Django thường bao gồm nhiều app, mỗi app chịu trách nhiệm cho một chức năng cụ thể.
- Nguyên tắc: **Tái sử dụng (reuse)** và **Đừng lặp lại chính mình (DRY - Don’t Repeat Yourself)**.
- Ví dụ: SuperBook có thể chia thành:
  - `accounts`: Quản lý tài khoản siêu anh hùng.
  - `posts`: Quản lý bài viết và bình luận.
  - `authentication`: Xác thực đăng nhập.

#### Ví dụ:
- Cấu trúc thư mục:
```
superbook/
├── manage.py
├── superbook/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── accounts/
│   ├── __init__.py
│   ├── models.py
│   ├── views.py
│   └── urls.py
└── posts/
    ├── __init__.py
    ├── models.py
    ├── views.py
    └── urls.py
```

### 6. Tái sử dụng hay tự làm? (Reuse or roll-your-own?)
#### Giải thích:
- Bạn có thể dùng các gói (package) có sẵn để tiết kiệm thời gian hoặc tự viết từ đầu nếu cần tùy chỉnh cao.
- Với SuperBook, bạn có thể dùng gói như `django-allauth` cho xác thực hoặc tự viết logic đăng nhập.

#### Ví dụ:
- Dùng gói có sẵn:
```bash
pip install django-allauth
```
- Thêm vào `settings.py`:
```python
INSTALLED_APPS = [
    ...
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
]
```
- Tự viết logic đăng nhập đơn giản:
```python
# accounts/views.py
from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login

def login_view(request):
    if request.method == "POST":
        username = request.POST["username"]
        password = request.POST["password"]
        user = authenticate(request, username=username, password=password)
        if user is not None:
            login(request, user)
            return redirect("home")
    return render(request, "accounts/login.html")
```

### 7. Các gói nào được chọn? (Which packages made it?)
#### Giải thích:
- Chọn gói dựa trên tính năng, độ phổ biến, và mức độ hỗ trợ.
- SuperBook dùng: `accounts`, `posts`, `authentication`.

### 8. Thực hành tốt nhất trước khi bắt đầu dự án (Best practices before starting a project)
#### Giải thích:
- Sử dụng môi trường ảo (virtual environment).
- Dùng hệ thống kiểm soát phiên bản (Git).
- Chọn mẫu dự án (project template) phù hợp.

#### Ví dụ:
- Tạo môi trường ảo:
```bash
python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows
```

### 9. SuperBook - Nhiệm vụ của bạn (SuperBook - your mission)
#### Giải thích:
- SuperBook là dự án mẫu xuyên suốt sách, minh họa cách áp dụng các pattern trong thực tế.

### 10. Tại sao dùng Python 3? (Why Python 3?)
#### Giải thích:
- Python 3 là phiên bản hiện đại, hỗ trợ tốt hơn Unicode và có nhiều cải tiến.
- Python 2 đã ngừng phát triển từ 2020.

### 11. Dùng phiên bản Django nào? (Which Django Version to use?)
#### Giải thích:
- Chọn phiên bản Long-Term Support (LTS) như Django 2.2 để ổn định, hoặc phiên bản mới nhất (như Django 4.x vào 2025) để có tính năng mới.

#### Ví dụ:
- Cài Django:
```bash
pip install django==2.2
```

### 12. Bắt đầu dự án (Starting the project)
#### Giải thích:
- Tạo dự án Django bằng lệnh `django-admin`.

#### Ví dụ:
```bash
django-admin startproject superbook
cd superbook
python manage.py startapp posts
python manage.py startapp accounts
```

---
