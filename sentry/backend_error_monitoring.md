### Tổng quan về Integrate Backend

Trang "Integrate Backend" cung cấp hướng dẫn cơ bản về cách cài đặt và cấu hình Sentry SDK (Software Development Kit) cho ứng dụng backend. Mục tiêu là giúp bạn:
- Gửi lỗi (errors) từ backend đến Sentry.
- Theo dõi hiệu suất (performance) của các giao dịch (transactions).
- Tích hợp Sentry vào quy trình phát triển phần mềm của bạn.

---

### Nội dung chính

#### 1. Tại sao cần tích hợp backend?
- **Mục đích**:
  - Backend là nơi xử lý logic chính của ứng dụng, nên việc giám sát lỗi và hiệu suất ở đây rất quan trọng.
  - Sentry giúp bạn phát hiện lỗi (như ngoại lệ - exceptions) và đo thời gian thực hiện của các tác vụ (như truy vấn cơ sở dữ liệu, gọi API).
- **Ví dụ thực tế**:
  - Một lỗi trong backend (như truy vấn SQL thất bại) có thể làm ứng dụng ngừng hoạt động. Sentry sẽ thông báo ngay để bạn sửa kịp thời.

#### 2. Các bước tích hợp cơ bản
Sentry hỗ trợ nhiều ngôn ngữ lập trình và framework cho backend (Python/Django, Node.js/Express, Java/Spring, v.v.). Dưới đây là các bước chung:

##### Bước 1: Cài đặt SDK
- **Mô tả**: Thêm thư viện Sentry SDK vào dự án của bạn.
- **Ví dụ với Python**:
  ```bash
  pip install sentry-sdk
  ```
- **Ví dụ với Node.js**:
  ```bash
  npm install @sentry/node
  ```

##### Bước 2: Khởi tạo SDK
- **Mô tả**: Thêm mã khởi tạo SDK vào ứng dụng với DSN (Data Source Name) từ Sentry.
- **Ví dụ với Python/Django**:
  ```python
  # settings.py
  import sentry_sdk
  from sentry_sdk.integrations.django import DjangoIntegration

  sentry_sdk.init(
      dsn="https://<your-dsn>@sentry.io/<project-id>",
      integrations=[DjangoIntegration()],
      traces_sample_rate=1.0  # Theo dõi 100% giao dịch
  )
  ```
- **Ví dụ với Node.js/Express**:
  ```javascript
  // server.js
  const Sentry = require("@sentry/node");
  const express = require("express");
  const app = express();

  Sentry.init({
      dsn: "https://<your-dsn>@sentry.io/<project-id>",
      traces_sample_rate: 1.0
  });

  app.use(Sentry.Handlers.requestHandler());  // Xử lý yêu cầu
  app.use(Sentry.Handlers.errorHandler());    // Xử lý lỗi
  ```

- **DSN**: Là chuỗi định danh bạn lấy từ Sentry khi tạo project, ví dụ: `https://abc123@sentry.io/456789`.

##### Bước 3: Kiểm tra tích hợp
- **Mô tả**: Gửi một lỗi thử nghiệm để đảm bảo Sentry nhận được dữ liệu.
- **Ví dụ Python**:
  ```python
  # views.py
  def test_sentry(request):
      raise Exception("Đây là lỗi thử nghiệm!")
  ```
- **Ví dụ Node.js**:
  ```javascript
  app.get("/test-sentry", (req, res) => {
      throw new Error("Đây là lỗi thử nghiệm!");
  });
  ```
- Sau khi chạy, lỗi sẽ xuất hiện trên dashboard Sentry.

#### 3. Tùy chỉnh cấu hình
- **Thêm ngữ cảnh (Context)**:
  - Gửi thông tin bổ sung như thông tin người dùng, môi trường (environment), hoặc tags.
  - Ví dụ Python:
    ```python
    sentry_sdk.set_user({"id": "123", "email": "user@example.com"})
    sentry_sdk.set_tag("environment", "production")
    ```
  - Ví dụ Node.js:
    ```javascript
    Sentry.setUser({ id: "123", email: "user@example.com" });
    Sentry.setTag("environment", "production");
    ```

- **Breadcrumbs**:
  - Ghi lại các sự kiện trước khi lỗi xảy ra (như truy vấn DB, yêu cầu HTTP).
  - Ví dụ Python:
    ```python
    sentry_sdk.add_breadcrumb(category="query", message="SELECT * FROM products")
    ```

- **Tỷ lệ lấy mẫu (Sample Rate)**:
  - Điều chỉnh `traces_sample_rate` để quyết định bao nhiêu phần trăm giao dịch được gửi đến Sentry (0.0 đến 1.0).
  - Ví dụ: `traces_sample_rate=0.5` theo dõi 50% giao dịch.

#### 4. Giám sát hiệu suất (Performance Monitoring)
- **Mô tả**: Theo dõi thời gian thực hiện của các tác vụ trong backend (như truy vấn cơ sở dữ liệu, gọi API bên ngoài).
- **Cách bật**:
  - Đặt `traces_sample_rate` trong cấu hình SDK.
- **Ví dụ thực tế**:
  - Đo thời gian từ khi nhận yêu cầu đến khi trả kết quả trong Django:
    ```python
    from sentry_sdk import start_transaction

    def product_view(request):
        with start_transaction(op="http.server", name="Product View"):
            products = Product.objects.all()
        return JsonResponse({"products": list(products.values())})
    ```

#### 5. Tích hợp với công cụ khác
- **Source Control (GitHub/GitLab)**:
  - Liên kết mã nguồn để xem commit gây lỗi.
  - Cấu hình trong Sentry: Settings > Integrations > GitHub.
- **Thông báo (Slack, Email)**:
  - Nhận cảnh báo khi lỗi xảy ra qua Slack hoặc email.

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
1. **Cài đặt và cấu hình**:
   ```python
   # settings.py
   import sentry_sdk
   from sentry_sdk.integrations.django import DjangoIntegration

   sentry_sdk.init(
       dsn="https://<your-dsn>@sentry.io/<project-id>",
       integrations=[DjangoIntegration()],
       traces_sample_rate=1.0,
       send_default_pii=True  # Gửi thông tin người dùng
   )
   ```

2. **Tạo lỗi thử nghiệm**:
   ```python
   # views.py
   from django.http import JsonResponse
   from decimal import Decimal

   def product_view(request):
       products = Product.objects.filter(price__lt=Decimal("50.00"))
       if not products:
           raise ValueError("Không tìm thấy sản phẩm dưới 50!")
       return JsonResponse({"products": list(products.values("name", "price"))})
   ```

3. **Kiểm tra trên Sentry**:
   - Truy cập `sentry.io`, vào project của bạn.
   - Tìm issue "ValueError: Không tìm thấy sản phẩm dưới 50!" với stack trace và thông tin ngữ cảnh (như URL, thời gian).

#### Đo hiệu suất
```python
# views.py
from sentry_sdk import start_transaction

def product_list(request):
    with start_transaction(op="http.server", name="Product List"):
        products = Product.objects.all()
        data = [{"name": p.name, "price": str(p.price)} for p in products]
    return JsonResponse({"products": data})
```
- Sentry sẽ ghi lại thời gian thực hiện toàn bộ giao dịch.

---

### Tóm tắt các bước tích hợp

| Bước                | Nội dung chính                                           |
|---------------------|---------------------------------------------------------|
| **Cài đặt SDK**     | Thêm thư viện Sentry vào dự án (pip, npm, v.v.).        |
| **Khởi tạo**        | Cấu hình SDK với DSN và các tùy chọn (traces, tags).    |
| **Kiểm tra**        | Gửi lỗi thử nghiệm để xác nhận tích hợp.                |
| **Tùy chỉnh**       | Thêm context, breadcrumbs, điều chỉnh sample rate.      |
| **Hiệu suất**       | Theo dõi giao dịch với `start_transaction`.             |