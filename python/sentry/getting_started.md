### Tổng quan về Sentry Basics

Sentry là một nền tảng giám sát ứng dụng giúp các nhà phát triển:
- Phát hiện và sửa lỗi (errors) trong ứng dụng.
- Theo dõi hiệu suất (performance) để đảm bảo ứng dụng chạy mượt mà.
- Học hỏi liên tục từ dữ liệu thực tế để cải thiện sản phẩm.

Trang "Sentry Basics" tập trung vào cách sử dụng Sentry để tìm kiếm, quản lý lỗi, và hiểu cách nó tích hợp vào quy trình phát triển phần mềm.

---

### Nội dung chính

#### 1. Tìm kiếm trong Sentry (Search)
- **Mô tả**: Sentry cho phép bạn tìm kiếm lỗi hoặc sự kiện (events) trong ứng dụng bằng cách sử dụng các bộ lọc (filters) và truy vấn (queries).
- **Cách hoạt động**:
  - Bạn có thể tìm kiếm dựa trên các thuộc tính như:
    - Tiêu đề lỗi (error title).
    - Tên người dùng (username).
    - Địa chỉ IP, thiết bị, hoặc phiên bản ứng dụng.
  - Ví dụ: Tìm tất cả lỗi có thông báo "NullPointerException" bằng cách nhập `NullPointerException` vào thanh tìm kiếm.
- **Tính năng nâng cao**:
  - **Loại trừ (Exclusion)**: Dùng dấu `!` để loại bỏ kết quả. Ví dụ: `!message:"timeout"` sẽ bỏ qua các lỗi có từ "timeout".
  - **Kết hợp điều kiện**: `is:unresolved user:john` tìm các lỗi chưa giải quyết của người dùng "john".
- **Ứng dụng thực tế**: Giúp bạn nhanh chóng xác định lỗi cụ thể trong hàng nghìn sự kiện.

#### 2. Khái niệm chính (Key Concepts)
- **Projects**: 
  - Mỗi ứng dụng hoặc thành phần trong hệ thống của bạn được gắn với một "project" trong Sentry.****
  - Ví dụ: Bạn có thể tạo project riêng cho frontend (React) và backend (Django).
- **Events**: 
  - Là các sự kiện được gửi đến Sentry, như lỗi (errors), giao dịch (transactions), hoặc thông điệp (messages).
  - Mỗi event chứa thông tin chi tiết: stack trace (dấu vết ngăn xếp), ngữ cảnh (context), thời gian xảy ra.
- **Issues**: 
  - Một "issue" là tập hợp các event tương tự, được nhóm lại dựa trên lỗi giống nhau.
  - Ví dụ: Nếu 100 người dùng gặp lỗi "404 Not Found" tại cùng một endpoint, Sentry nhóm chúng thành một issue.
- **Releases**: 
  - Liên kết lỗi với phiên bản cụ thể của ứng dụng (release version).
  - Giúp bạn biết lỗi xuất hiện từ phiên bản nào (ví dụ: "v1.2.3").

#### 3. Tích hợp và cấu hình (Integration & Configuration)
- **SDK (Software Development Kit)**:
  - Sentry cung cấp các SDK cho nhiều ngôn ngữ/framework (JavaScript, Python, Java, v.v.).
  - Bạn thêm SDK vào mã nguồn để gửi lỗi và dữ liệu hiệu suất đến Sentry.
  - Ví dụ với Python/Django:
    ```python
    import sentry_sdk
    from sentry_sdk.integrations.django import DjangoIntegration

    sentry_sdk.init(
        dsn="https://<your-dsn>@sentry.io/<project-id>",
        integrations=[DjangoIntegration()],
        traces_sample_rate=1.0  # Theo dõi 100% giao dịch
    )
    ```
- **DSN (Data Source Name)**:
  - Là một chuỗi định danh duy nhất bạn lấy từ Sentry để kết nối ứng dụng với project.
  - Ví dụ: `https://abc123@sentry.io/456789`.

#### 4. Quản lý lỗi (Error Management)
- **Xem chi tiết lỗi**:
  - Khi một lỗi xảy ra, Sentry hiển thị:
    - Stack trace: Dòng mã gây lỗi.
    - Context: Thông tin người dùng, thiết bị, URL.
    - Tags: Các nhãn tùy chỉnh (ví dụ: "environment=production").
- **Trạng thái lỗi**:
  - **Unresolved**: Lỗi chưa được xử lý.
  - **Resolved**: Đánh dấu khi lỗi đã sửa.
  - **Ignored**: Bỏ qua lỗi không quan trọng.
- **Gán lỗi**: Gán issue cho thành viên trong nhóm để xử lý.

#### 5. Giám sát hiệu suất (Performance Monitoring)
- **Transactions**:
  - Theo dõi thời gian thực hiện của các giao dịch (ví dụ: tải trang, gọi API).
  - Ví dụ: Đo thời gian từ khi người dùng nhấp nút đến khi server trả về dữ liệu.
- **Traces**:
  - Kết nối các sự kiện liên quan (frontend → backend) để tìm điểm chậm (bottleneck).
- **Tỷ lệ lấy mẫu (Sample Rate)**:
  - Điều chỉnh `traces_sample_rate` (0.0 đến 1.0) để quyết định bao nhiêu phần trăm giao dịch được theo dõi.

#### 6. Tùy chỉnh và nâng cao
- **Tags**: Thêm nhãn để phân loại lỗi (ví dụ: `tag: {"env": "prod", "version": "1.0"}`).
- **Breadcrumbs**: Ghi lại hành động người dùng trước khi lỗi xảy ra (như nhấp chuột, gửi form).
- **User Feedback**: Cho phép người dùng báo cáo lỗi trực tiếp từ ứng dụng.

---

### Ví dụ thực tế với Product Model

#### Model
```python
# myapp/models.py
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=255)
    product_key = models.CharField(max_length=50, unique=True)
    category = models.CharField(max_length=100)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField()
    created_at = models.DateTimeField(auto_now_add=True)
```

#### Tích hợp Sentry
1. **Cài đặt SDK**:
   ```bash
   pip install sentry-sdk
   ```
2. **Cấu hình trong settings.py**:
   ```python
   # myproject/settings.py
   import sentry_sdk
   from sentry_sdk.integrations.django import DjangoIntegration

   sentry_sdk.init(
       dsn="https://<your-dsn>@sentry.io/<project-id>",
       integrations=[DjangoIntegration()],
       traces_sample_rate=1.0,
       send_default_pii=True  # Gửi thông tin người dùng (nếu có)
   )
   ```
3. **Thêm lỗi mẫu**:
   ```python
   # myapp/views.py
   from django.http import JsonResponse
   from decimal import Decimal

   def product_view(request):
       products = Product.objects.filter(price__lt=Decimal("50.00"))
       if not products:
           raise ValueError("Không tìm thấy sản phẩm dưới 50!")
       return JsonResponse({"products": list(products.values("name", "price"))})
   ```
   - Nếu không có sản phẩm nào, Sentry sẽ ghi lại lỗi `ValueError`.

4. **Kiểm tra trong Sentry**:
   - Truy cập `sentry.io`, vào project của bạn.
   - Tìm issue "ValueError: Không tìm thấy sản phẩm dưới 50!" với stack trace chỉ đến dòng `raise ValueError`.

---

### Tóm tắt

| Phần                | Nội dung chính                                                                 |
|---------------------|-------------------------------------------------------------------------------|
| **Tìm kiếm**        | Dùng bộ lọc và truy vấn để tìm lỗi cụ thể.                                    |
| **Khái niệm**       | Projects, Events, Issues, Releases là các thành phần cơ bản.                  |
| **Tích hợp**        | Dùng SDK và DSN để kết nối ứng dụng với Sentry.                               |
| **Quản lý lỗi**     | Xem chi tiết, đánh dấu trạng thái, gán lỗi cho đội ngũ.                       |
| **Hiệu suất**       | Theo dõi giao dịch, traces, và điều chỉnh tỷ lệ lấy mẫu.                      |
| **Tùy chỉnh**       | Thêm tags, breadcrumbs, và lấy phản hồi từ người dùng.                        |
