### Tổng quan về Sentry Django Integration

Trang này tập trung vào việc tích hợp **Sentry SDK** vào dự án Django sử dụng Python. Mục tiêu là:
- Gửi lỗi từ ứng dụng Django đến Sentry (như ngoại lệ, exception).
- Theo dõi hiệu suất của các giao dịch (HTTP requests, truy vấn cơ sở dữ liệu).
- Cung cấp thông tin chi tiết để debug và cải thiện ứng dụng.

---

### Nội dung chính

#### 1. Cài đặt (Getting Started)
##### Bước 1: Cài đặt SDK
- **Mô tả**: Thêm thư viện `sentry-sdk` vào dự án Django.
- **Lệnh cài đặt**:
  ```bash
  pip install --upgrade sentry-sdk
  ```

##### Bước 2: Cấu hình trong Django
- **Mô tả**: Thêm mã khởi tạo Sentry vào file cấu hình của Django (`settings.py`).
- **Ví dụ**:
  ```python
  # myproject/settings.py
  import sentry_sdk
  from sentry_sdk.integrations.django import DjangoIntegration

  sentry_sdk.init(
      dsn="https://<your-dsn>@sentry.io/<project-id>",
      integrations=[
          DjangoIntegration(),
      ],
      traces_sample_rate=1.0,  # Theo dõi 100% giao dịch
      send_default_pii=True    # Gửi thông tin người dùng (nếu có)
  )
  ```
- **Giải thích**:
  - `dsn`: Chuỗi định danh lấy từ Sentry khi tạo project (ví dụ: `https://abc123@sentry.io/456789`).
  - `DjangoIntegration()`: Tích hợp đặc biệt cho Django, tự động thu thập lỗi và hiệu suất từ middleware, views, v.v.
  - `traces_sample_rate`: Tỷ lệ lấy mẫu giao dịch (1.0 = 100%, 0.5 = 50%).
  - `send_default_pii`: Gửi thông tin cá nhân (Personally Identifiable Information) như IP, username (nếu bật).

##### Bước 3: Kiểm tra tích hợp
- **Mô tả**: Gửi một lỗi thử nghiệm để xác nhận Sentry hoạt động.
- **Ví dụ**:
  ```python
  # myapp/views.py
  from django.http import HttpResponse

  def test_sentry(request):
      raise Exception("Đây là lỗi thử nghiệm trong Django!")
      return HttpResponse("Không bao giờ đến đây!")
  ```
- **Kết quả**: Truy cập URL (như `/test-sentry/`), lỗi sẽ xuất hiện trên dashboard Sentry với stack trace.

#### 2. Giám sát hiệu suất (Performance Monitoring)
- **Mô tả**: Theo dõi thời gian thực hiện của các giao dịch trong Django (HTTP requests, truy vấn DB).
- **Cách bật**: Đặt `traces_sample_rate` trong `sentry_sdk.init()`.
- **Ví dụ**:
  ```python
  # myapp/views.py
  from django.http import JsonResponse
  from sentry_sdk import start_transaction

  def product_list(request):
      with start_transaction(op="http.server", name="Product List"):
          products = Product.objects.all()
          data = [{"name": p.name} for p in products]
      return JsonResponse({"products": data})
  ```
- **Tự động thu thập**:
  - `DjangoIntegration` tự động theo dõi các middleware, truy vấn cơ sở dữ liệu (qua Django ORM), và template rendering.
- **Kết quả trên Sentry**:
  - Thời gian xử lý request (ví dụ: 200ms).
  - Chi tiết từng bước: middleware (50ms), DB query (100ms), v.v.

#### 3. Tùy chỉnh (Configuration Options)
- **Thêm ngữ cảnh (Context)**:
  - Gửi thông tin bổ sung như người dùng, tags.
  - Ví dụ:
    ```python
    # views.py
    from sentry_sdk import set_user, set_tag

    def user_view(request):
        set_user({"id": request.user.id, "email": request.user.email})
        set_tag("environment", "production")
        raise ValueError("Lỗi từ người dùng!")
  ```
- **Breadcrumbs**:
  - Ghi lại hành động trước lỗi (như truy vấn DB, request).
  - Ví dụ:
    ```python
    from sentry_sdk import add_breadcrumb

    def product_view(request):
        add_breadcrumb(category="db", message="Truy vấn tất cả sản phẩm")
        products = Product.objects.all()
        return JsonResponse({"products": list(products.values())})
    ```

- **Lọc sự kiện (Event Filtering)**:
  - Loại bỏ lỗi không quan trọng trước khi gửi đến Sentry.
  - Ví dụ:
    ```python
    # settings.py
    def before_send(event, hint):
        if "Timeout" in str(event.get("exception", "")):
            return None  # Bỏ qua lỗi Timeout
        return event

    sentry_sdk.init(
        dsn="https://<your-dsn>@sentry.io/<project-id>",
        integrations=[DjangoIntegration()],
        before_send=before_send
    )
    ```

#### 4. Xử lý lỗi ngoại lệ (Exception Handling)
- **Tự động bắt lỗi**:
  - `DjangoIntegration` tự động thu thập các ngoại lệ từ:
    - Views (như `ValueError`, `Http404`).
    - Middleware.
    - Template errors (như biến không tồn tại).
- **Bắt lỗi thủ công**:
  - Ví dụ:
    ```python
    from sentry_sdk import capture_exception

    def risky_view(request):
        try:
            result = 1 / 0
        except ZeroDivisionError as e:
            capture_exception(e)  # Gửi lỗi đến Sentry
        return HttpResponse("Đã xử lý lỗi!")
    ```

#### 5. Ví dụ thực tế với Product Model
##### Model
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

##### Tích hợp Sentry
1. **Cấu hình**:
   ```python
   # settings.py
   import sentry_sdk
   from sentry_sdk.integrations.django import DjangoIntegration

   sentry_sdk.init(
       dsn="https://<your-dsn>@sentry.io/<project-id>",
       integrations=[DjangoIntegration()],
       traces_sample_rate=1.0,
       send_default_pii=True
   )
   ```

2. **View với lỗi và hiệu suất**:
   ```python
   # myapp/views.py
   from django.http import JsonResponse
   from sentry_sdk import start_transaction, add_breadcrumb
   from decimal import Decimal

   def product_list(request):
       with start_transaction(op="http.server", name="Product List"):
           add_breadcrumb(category="db", message="Lọc sản phẩm dưới 50")
           products = Product.objects.filter(price__lt=Decimal("50.00"))
           if not products:
               raise ValueError("Không tìm thấy sản phẩm dưới 50!")
           data = [{"name": p.name, "price": str(p.price)} for p in products]
       return JsonResponse({"products": data})
   ```

3. **Kiểm tra trên Sentry**:
   - Truy cập URL (như `/products/`).
   - Nếu không có sản phẩm, Sentry ghi lại lỗi `ValueError`.
   - Nếu có sản phẩm, Sentry hiển thị giao dịch "Product List" với thời gian xử lý.

#### 6. Các tính năng khác
- **Source Maps**: Nếu dùng Django với frontend (như Django Templates + JS), Sentry hỗ trợ source maps để debug mã JS.
- **Logging Integration**: Kết hợp với Python `logging`:
  ```python
  import logging
  from sentry_sdk.integrations.logging import LoggingIntegration

  sentry_sdk.init(
      dsn="https://<your-dsn>@sentry.io/<project-id>",
      integrations=[DjangoIntegration(), LoggingIntegration(level=logging.ERROR)]
  )

  logger = logging.getLogger(__name__)
  logger.error("Đây là lỗi từ logging!")
  ```

---

### Tóm tắt

| Phần                  | Nội dung chính                                                      |
|-----------------------|--------------------------------------------------------------------|
| **Cài đặt**           | Thêm `sentry-sdk`, cấu hình trong `settings.py` với `DjangoIntegration`. |
| **Hiệu suất**         | Theo dõi giao dịch tự động hoặc thủ công với `start_transaction`.   |
| **Tùy chỉnh**         | Thêm context, breadcrumbs, lọc sự kiện qua `before_send`.          |
| **Xử lý lỗi**         | Tự động bắt lỗi từ views/middleware hoặc thủ công với `capture_exception`. |
| **Ví dụ**             | Áp dụng vào `Product` model để giám sát lỗi và hiệu suất.          |