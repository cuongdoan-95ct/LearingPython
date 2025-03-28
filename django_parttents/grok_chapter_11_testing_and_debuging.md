### Chương 11: Kiểm thử và Gỡ lỗi (Tổng quan)

Chương này tập trung vào hai khía cạnh quan trọng trong phát triển phần mềm với Django: **kiểm thử (testing)** và **gỡ lỗi (debugging)**. Nó giải thích tại sao cần kiểm thử, cách viết kiểm thử hiệu quả, và các công cụ gỡ lỗi giúp xác định và sửa lỗi trong mã nguồn.

#### Nội dung chính:
1. **Tại sao cần viết kiểm thử?**
2. **Phát triển theo hướng kiểm thử (TDD)**
3. **Viết một trường hợp kiểm thử**
4. **Mocking (Giả lập)**
5. **Mẫu kiểm thử: Test Fixtures và Factories**
6. **Gỡ lỗi trong Django**

---

### 1. Tại sao cần viết kiểm thử? (Why write tests?)

Kiểm thử đảm bảo mã hoạt động đúng như mong đợi, đặc biệt khi dự án phát triển hoặc thay đổi. Không có kiểm thử, việc bảo trì hoặc thêm tính năng mới có thể dẫn đến lỗi không mong muốn.

#### Lợi ích:
- **Phát hiện lỗi sớm**: Tránh lỗi lan rộng trong hệ thống.
- **Tài liệu hóa mã**: Kiểm thử như một tài liệu sống, mô tả cách mã nên hoạt động.
- **Tự tin khi thay đổi**: Có thể tái cấu trúc (refactor) mà không sợ phá hỏng chức năng.

#### Ví dụ thực tế:
Nếu bạn thêm một tính năng vào ứng dụng mà không kiểm thử, một lỗi nhỏ (như sai logic tính toán) có thể không được phát hiện cho đến khi người dùng phàn nàn.

---

### 2. Phát triển theo hướng kiểm thử (TDD - Test-Driven Development)

**TDD** là phương pháp viết kiểm thử trước khi viết mã thực tế. Quy trình TDD bao gồm:
1. Viết một kiểm thử thất bại (red).
2. Viết mã tối thiểu để kiểm thử qua (green).
3. Tái cấu trúc mã mà vẫn đảm bảo kiểm thử qua (refactor).

#### Ví dụ TDD:
Giả sử bạn xây dựng hàm tính tổng giỏ hàng:
```python
# tests.py
from django.test import TestCase
from myapp.utils import tinh_tong_gio_hang

class TestGioHang(TestCase):
    def test_tinh_tong_gio_hang(self):
        gio_hang = [{"gia": 100}, {"gia": 200}]
        self.assertEqual(tinh_tong_gio_hang(gio_hang), 300)
```
- Chạy kiểm thử: Thất bại vì hàm chưa tồn tại.
- Viết mã:
```python
# utils.py
def tinh_tong_gio_hang(gio_hang):
    return sum(item["gia"] for item in gio_hang)
```
- Chạy lại: Thành công (green). Sau đó, tái cấu trúc nếu cần.

---

### 3. Viết một trường hợp kiểm thử (Writing a test case)

Django cung cấp lớp `TestCase` để viết kiểm thử. Phương thức `assert` được dùng để kiểm tra kết quả.

#### Các bước:
- Tạo lớp kiểm thử kế thừa `django.test.TestCase`.
- Viết phương thức bắt đầu bằng `test_`.
- Dùng `assert` để so sánh kết quả thực tế với kỳ vọng.

#### Ví dụ cơ bản:
```python
# tests.py
from django.test import TestCase
from myapp.models import SanPham

class TestSanPham(TestCase):
    def test_tao_san_pham(self):
        sp = SanPham.objects.create(ten="iPhone", gia=1000)
        self.assertEqual(sp.ten, "iPhone")
        self.assertEqual(sp.gia, 1000)
```
- Kiểm tra xem sản phẩm được tạo đúng chưa.

#### Phương thức assert phổ biến:
- `assertEqual(a, b)`: Kiểm tra a == b.
- `assertTrue(x)`: Kiểm tra x là True.
- `assertRaises(Exception, func)`: Kiểm tra hàm có抛出 (ném) ngoại lệ không.

#### Viết kiểm thử tốt hơn:
- **Đừng lặp lại mã**: Sử dụng `setUp` để khởi tạo dữ liệu chung.
```python
class TestSanPham(TestCase):
    def setUp(self):
        self.sp = SanPham.objects.create(ten="iPhone", gia=1000)

    def test_ten_san_pham(self):
        self.assertEqual(self.sp.ten, "iPhone")

    def test_gia_san_pham(self):
        self.assertEqual(self.sp.gia, 1000)
```
- Tránh kiểm thử quá phức tạp hoặc không cần thiết.

---

### 4. Mocking (Giả lập)

**Mocking** thay thế các đối tượng thực (như API bên ngoài, database) bằng các đối tượng giả để kiểm thử độc lập.

#### Ví dụ:
Giả lập gửi email:
```python
# utils.py
from django.core.mail import send_mail

def gui_thong_bao(email, noi_dung):
    send_mail("Thông báo", noi_dung, "from@example.com", [email])
```
```python
# tests.py
from django.test import TestCase
from unittest.mock import patch
from myapp.utils import gui_thong_bao

class TestThongBao(TestCase):
    @patch("myapp.utils.send_mail")  # Giả lập send_mail
    def test_gui_thong_bao(self, mock_send_mail):
        gui_thong_bao("user@example.com", "Chào mừng!")
        mock_send_mail.assert_called_with(
            "Thông báo", "Chào mừng!", "from@example.com", ["user@example.com"]
        )
```
- `patch` thay `send_mail` bằng một mock, kiểm tra xem hàm được gọi đúng tham số chưa.

---

### 5. Mẫu kiểm thử: Test Fixtures và Factories (Pattern - Test Fixtures and Factories)

#### Vấn đề:
- Tạo dữ liệu kiểm thử thủ công (như trong `setUp`) tốn thời gian và khó bảo trì khi dữ liệu phức tạp.

#### Giải pháp:
- **Fixtures**: Tệp JSON chứa dữ liệu mẫu.
- **Factories**: Công cụ tạo dữ liệu động (như `factory_boy`).

##### Ví dụ Fixtures:
```json
# fixtures/sanpham.json
[
    {
        "model": "myapp.sanpham",
        "pk": 1,
        "fields": {
            "ten": "iPhone",
            "gia": 1000
        }
    }
]
```
```python
# tests.py
class TestSanPham(TestCase):
    fixtures = ["sanpham.json"]

    def test_san_pham_tu_fixture(self):
        sp = SanPham.objects.get(pk=1)
        self.assertEqual(sp.ten, "iPhone")
```
- Tải dữ liệu từ `sanpham.json` vào database trước khi kiểm thử.

##### Ví dụ Factories:
Cài đặt: `pip install factory-boy`
```python
# factories.py
import factory
from myapp.models import SanPham

class SanPhamFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = SanPham

    ten = factory.Faker("word")  # Tên ngẫu nhiên
    gia = factory.Faker("random_int", min=100, max=1000)
```
```python
# tests.py
from django.test import TestCase
from .factories import SanPhamFactory

class TestSanPham(TestCase):
    def test_san_pham_factory(self):
        sp = SanPhamFactory()  # Tạo sản phẩm ngẫu nhiên
        self.assertTrue(isinstance(sp.gia, int))
        self.assertTrue(100 <= sp.gia <= 1000)
```
- `factory_boy` tạo dữ liệu động, dễ mở rộng hơn fixtures.

#### Khi nào dùng?
- **Fixtures**: Dữ liệu cố định, ít thay đổi.
- **Factories**: Dữ liệu phức tạp, cần linh hoạt.

---

### 6. Gỡ lỗi trong Django (Debugging)

Khi mã có lỗi, cần công cụ để tìm và sửa.

#### a. Trang gỡ lỗi Django (Django debug page)
- Khi `DEBUG = True` trong `settings.py`, Django hiển thị trang chi tiết khi có lỗi:
  - **Traceback**: Nơi lỗi xảy ra.
  - **Thông tin request**: Dữ liệu gửi đến server.
  - **Chi tiết ngoại lệ**: Nguyên nhân lỗi.
- Ví dụ: Truy cập URL không tồn tại → 404 với stack trace.

#### b. Trang gỡ lỗi cải tiến
- Dùng `django-extensions` để thêm thông tin:
```bash
pip install django-extensions
```
```python
# settings.py
INSTALLED_APPS += ["django_extensions"]
```
- Chạy: `python manage.py runserver_plus` → Trang lỗi chi tiết hơn.

#### c. Hàm print
- In giá trị biến để kiểm tra:
```python
# views.py
def danh_sach_san_pham(request):
    san_pham = SanPham.objects.all()
    print("Số sản phẩm:", san_pham.count())
    return render(request, "danh_sach.html", {"san_pham": san_pham})
```

#### d. Logging
- Ghi log thay vì `print`:
```python
# settings.py
LOGGING = {
    "version": 1,
    "handlers": {
        "console": {"class": "logging.StreamHandler"}
    },
    "loggers": {
        "myapp": {"handlers": ["console"], "level": "DEBUG"}
    }
}
```
```python
# views.py
import logging
logger = logging.getLogger("myapp")

def danh_sach_san_pham(request):
    san_pham = SanPham.objects.all()
    logger.debug(f"Số sản phẩm: {san_pham.count()}")
    return render(request, "danh_sach.html", {"san_pham": san_pham})
```
- Log lưu trữ lâu dài, dễ theo dõi hơn `print`.

#### e. Django Debug Toolbar
- Cài đặt: `pip install django-debug-toolbar`
```python
# settings.py
INSTALLED_APPS += ["debug_toolbar"]
MIDDLEWARE += ["debug_toolbar.middleware.DebugToolbarMiddleware"]
INTERNAL_IPS = ["127.0.0.1"]
```
- Hiển thị thanh công cụ trên trang web, cho biết thời gian truy vấn SQL, cache, v.v.

#### f. Python Debugger (pdb)
- Dừng mã để kiểm tra:
```python
# views.py
import pdb

def danh_sach_san_pham(request):
    san_pham = SanPham.objects.all()
    pdb.set_trace()  # Dừng tại đây
    return render(request, "danh_sach.html", {"san_pham": san_pham})
```
- Khi chạy, dùng lệnh:
  - `n` (next): Chạy dòng tiếp theo.
  - `p variable`: In giá trị biến.
  - `q` (quit): Thoát.

#### g. Gỡ lỗi template
- Kiểm tra lỗi trong template:
```python
# settings.py
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "OPTIONS": {"debug": True},
    }
]
```
- Khi có lỗi (như biến không tồn tại), Django hiển thị chi tiết lỗi template.

---

### Tóm tắt
- **Tại sao kiểm thử**: Phát hiện lỗi, tài liệu hóa, tự tin thay đổi.
- **TDD**: Viết kiểm thử trước, mã sau, tái cấu trúc.
- **Kiểm thử**: Dùng `TestCase`, `assert`, tối ưu với `setUp`.
- **Mocking**: Giả lập để kiểm tra độc lập.
- **Fixtures/Factories**: Dữ liệu mẫu cố định hoặc động.
- **Gỡ lỗi**: Dùng trang debug, print, logging, Debug Toolbar, pdb.