Để debug hiệu suất truy vấn SQL trong Django, bạn cần theo dõi và phân tích các truy vấn SQL được thực thi, xác định những truy vấn chậm, dư thừa, hoặc không tối ưu, từ đó cải thiện hiệu suất ứng dụng. Dưới đây là hướng dẫn chi tiết bằng tiếng Việt về cách debug hiệu suất truy vấn SQL trong Django, sử dụng các công cụ và kỹ thuật phổ biến, phù hợp với ngữ cảnh ứng dụng Django (ví dụ: `ProductViewSet` từ các trao đổi trước).

---

### **1. Mục tiêu khi debug hiệu suất SQL**
- **Xác định số lượng truy vấn**: Kiểm tra xem có quá nhiều truy vấn không cần thiết (ví dụ: N+1 query problem).
- **Đo thời gian thực thi**: Tìm các truy vấn chậm hoặc tốn tài nguyên.
- **Kiểm tra tối ưu hóa**: Đảm bảo truy vấn sử dụng index, không lặp lại, và tận dụng cache khi cần.
- **Tích hợp với cache**: Kiểm tra xem cache (như Redis trong trường hợp của bạn) có giảm số lượng truy vấn SQL hay không.

---

### **2. Các công cụ và kỹ thuật debug hiệu suất SQL**

#### **Bước 1: Sử dụng Django Debug Toolbar**
**Django Debug Toolbar** là công cụ mạnh mẽ để phân tích truy vấn SQL và hiệu suất tổng thể.

- **Cài đặt**:
  ```bash
  pip install django-debug-toolbar
  ```

- **Cấu hình** trong `settings.py`:
  ```python
  INSTALLED_APPS = [
      # ...
      'debug_toolbar',
  ]

  MIDDLEWARE = [
      # ...
      'debug_toolbar.middleware.DebugToolbarMiddleware',
  ]

  INTERNAL_IPS = ['127.0.0.1']

  # Tùy chọn: Hiển thị tất cả truy vấn SQL
  DEBUG_TOOLBAR_PANELS = [
      'debug_toolbar.panels.versions.VersionsPanel',
      'debug_toolbar.panels.timer.TimerPanel',
      'debug_toolbar.panels.settings.SettingsPanel',
      'debug_toolbar.panels.headers.HeadersPanel',
      'debug_toolbar.panels.request.RequestPanel',
      'debug_toolbar.panels.sql.SQLPanel',  # Quan trọng để debug SQL
      'debug_toolbar.panels.staticfiles.StaticFilesPanel',
      'debug_toolbar.panels.templates.TemplatesPanel',
      'debug_toolbar.panels.cache.CachePanel',  # Để kiểm tra cache
      'debug_toolbar.panels.signals.SignalsPanel',
      'debug_toolbar.panels.logging.LoggingPanel',
      'debug_toolbar.panels.redirects.RedirectsPanel',
  ]
  ```

- **Cách sử dụng**:
  - Truy cập ứng dụng (ví dụ: `/api/products/` hoặc trang HTML) trong trình duyệt với `DEBUG = True`.
  - Thanh toolbar sẽ xuất hiện ở bên phải màn hình.
  - Nhấp vào tab **SQL** để xem:
    - **Danh sách truy vấn SQL**: Mỗi truy vấn được liệt kê cùng thời gian thực thi (tính bằng mili giây).
    - **Số lượng truy vấn**: Tổng số truy vấn cho trang.
    - **Chi tiết truy vấn**: Nhấp vào truy vấn để xem câu lệnh SQL đầy đủ và kế hoạch thực thi (EXPLAIN).
    - **Truy vấn trùng lặp**: Toolbar đánh dấu các truy vấn giống nhau, giúp phát hiện lặp dư thừa.

- **Debug hiệu suất**:
  - Nếu số truy vấn quá cao (ví dụ: >50 cho một trang đơn giản), kiểm tra xem có vấn đề N+1 không (xem bên dưới).
  - Nếu thời gian thực thi của một truy vấn dài (>100ms), chạy `EXPLAIN` để phân tích.

#### **Bước 2: Kiểm tra vấn đề N+1 Query**
Vấn đề N+1 xảy ra khi Django thực hiện một truy vấn để lấy danh sách đối tượng, sau đó thực hiện thêm truy vấn riêng cho mỗi đối tượng (thường do truy cập trường liên quan).

- **Ví dụ vấn đề N+1**:
  Trong `ProductViewSet`, nếu bạn có:
  ```python
  class ProductViewSet(viewsets.ModelViewSet):
      queryset = Product.objects.all()
      serializer_class = ProductSerializer
  ```
  Và serializer truy cập một trường liên quan (như ForeignKey):
  ```python
  class ProductSerializer(serializers.ModelSerializer):
      category = CategorySerializer()  # Category là ForeignKey
      class Meta:
          model = Product
          fields = ['id', 'name', 'price', 'category']
  ```
  Django sẽ thực hiện 1 truy vấn để lấy danh sách sản phẩm, sau đó thực hiện N truy vấn để lấy thông tin danh mục cho mỗi sản phẩm.

- **Cách debug**:
  - Sử dụng Django Debug Toolbar để xem số lượng truy vấn. Nếu số truy vấn tăng tuyến tính với số sản phẩm, đó là dấu hiệu N+1.
  - Dùng `django.db.connection` để ghi lại truy vấn:
    ```python
    from django.db import connection

    def list(self, request, *args, **kwargs):
        queryset = self.get_queryset()
        serializer = self.get_serializer(queryset, many=True)
        print(len(connection.queries))  # Số lượng truy vấn
        for query in connection.queries:
            print(query['sql'], query['time'])  # In truy vấn và thời gian
        return Response(serializer.data)
    ```

- **Cách khắc phục**:
  - Sử dụng `select_related` cho ForeignKey hoặc OneToOneField:
    ```python
    class ProductViewSet(viewsets.ModelViewSet):
        queryset = Product.objects.select_related('category').all()
        serializer_class = ProductSerializer
    ```
    - `select_related` thực hiện JOIN để lấy dữ liệu liên quan trong một truy vấn.
  - Sử dụng `prefetch_related` cho ManyToManyField hoặc ForeignKey ngược:
    ```python
    queryset = Product.objects.prefetch_related('tags').all()
    ```
    - `prefetch_related` lấy dữ liệu liên quan trong truy vấn riêng nhưng tối ưu hơn.
  - Kiểm tra lại bằng Debug Toolbar để đảm bảo số truy vấn giảm.

#### **Bước 3: Sử dụng `EXPLAIN` để phân tích truy vấn chậm**
Nếu một truy vấn mất quá nhiều thời gian (ví dụ: >100ms trong Debug Toolbar), bạn có thể phân tích nó bằng `EXPLAIN`.

- **Cách thực hiện**:
  - Lấy câu lệnh SQL từ Debug Toolbar hoặc `connection.queries`.
  - Chạy `EXPLAIN` trong công cụ quản lý cơ sở dữ liệu (như psql cho PostgreSQL):
    ```sql
    EXPLAIN ANALYZE SELECT * FROM myapp_product WHERE price > 100;
    ```
    - `EXPLAIN ANALYZE` hiển thị kế hoạch thực thi và thời gian thực tế.
    - Tìm các điểm như:
      - **Seq Scan**: Quét toàn bảng, có thể cần index.
      - **Cost cao**: Truy vấn tốn tài nguyên.
      - **Rows**: Số hàng thực tế so với dự đoán.

- **Ví dụ trong Django**:
  ```python
  from django.db import connection

  with connection.cursor() as cursor:
      cursor.execute('EXPLAIN ANALYZE SELECT * FROM myapp_product WHERE price > 100')
      result = cursor.fetchall()
      for row in result:
          print(row)
  ```

- **Tối ưu hóa**:
  - Thêm index cho cột thường xuyên được lọc hoặc sắp xếp:
    ```python
    class Product(models.Model):
        name = models.CharField(max_length=100)
        price = models.DecimalField(max_digits=10, decimal_places=2)

        class Meta:
            indexes = [
                models.Index(fields=['price']),
            ]
    ```
    Chạy migration:
    ```bash
    python manage.py makemigrations
    python manage.py migrate
    ```
  - Giảm số cột trong `SELECT` bằng `only` hoặc `defer`:
    ```python
    Product.objects.only('name', 'price').all()  # Chỉ lấy các cột cần thiết
    ```

#### **Bước 4: Sử dụng Django Shell để kiểm tra truy vấn**
Django shell cho phép bạn chạy truy vấn và kiểm tra hiệu suất trực tiếp.

- **Ví dụ**:
  ```bash
  python manage.py shell
  ```
  ```python
  from django.db import connection
  from myapp.models import Product

  # Kiểm tra truy vấn
  products = Product.objects.all()
  print(products.query)  # In câu lệnh SQL
  print(len(connection.queries))  # Số lượng truy vấn

  # Reset truy vấn để đo lại
  from django.db import reset_queries
  reset_queries()

  # Test N+1
  for product in Product.objects.all():
      print(product.category.name)  # Gây N+1
  print(len(connection.queries))  # Số truy vấn cao

  # Test với select_related
  reset_queries()
  for product in Product.objects.select_related('category'):
      print(product.category.name)
  print(len(connection.queries))  # Số truy vấn giảm
  ```

#### **Bước 5: Tích hợp với cache (Redis)**
Vì bạn sử dụng Redis làm backend cache, hãy kiểm tra xem cache có giảm số truy vấn SQL hay không.

- **Kiểm tra cache hit/miss**:
  Trong `ProductViewSet`, logging đã được thêm:
  ```python
  def retrieve(self, request, *args, **kwargs):
      product_id = kwargs['pk']
      cache_key = f'product_{product_id}'
      product_data = cache.get(cache_key)
      if product_data:
          logger.info(f"Cache hit for key: {cache_key}")
      else:
          logger.info(f"Cache miss for key: {cache_key}")
          instance = self.get_object()
          serializer = self.get_serializer(instance)
          product_data = serializer.data
          cache.set(cache_key, product_data, 60 * 5)
          logger.info(f"Set cache for key: {cache_key}, expires in 5 minutes")
      return Response(product_data)
  ```
  - Nếu thấy "Cache hit" trong `debug.log` và số truy vấn SQL (trong Debug Toolbar) là 0, cache đang hoạt động tốt.
  - Nếu thấy "Cache miss" và số truy vấn cao, kiểm tra logic cache hoặc TTL.

- **Kiểm tra Redis**:
  ```bash
  redis-cli -n 1
  KEYS myapp:*
  GET myapp:product_1
  TTL myapp:product_1
  ```

#### **Bước 6: Sử dụng công cụ bên ngoài**
- **PostgreSQL/MySQL**:
  - Kích hoạt slow query log để ghi lại các truy vấn chậm:
    - PostgreSQL: Sửa `postgresql.conf`:
      ```conf
      log_min_duration_statement = 100  # Ghi lại truy vấn >100ms
      ```
    - MySQL: Sửa `my.cnf`:
      ```conf
      slow_query_log = 1
      long_query_time = 0.1  # Ghi lại truy vấn >100ms
      ```
  - Phân tích log bằng `pt-query-digest` (cho MySQL) hoặc `pgBadger` (cho PostgreSQL).
- **New Relic/Datadog**:
  - Trong môi trường production, sử dụng các công cụ này để theo dõi hiệu suất SQL và nhận cảnh báo về truy vấn chậm.
- **django-silk**:
  - Một thay thế cho Django Debug Toolbar, ghi lại tất cả truy vấn và hiệu suất:
    ```bash
    pip install django-silk
    ```
    ```python
    INSTALLED_APPS = ['silk']
    MIDDLEWARE = ['silk.middleware.SilkyMiddleware']
    ```
    Truy cập `/silk/` để xem báo cáo truy vấn.

---

### **3. Ví dụ thực tế với ProductViewSet**
Dựa trên `ProductViewSet`, dưới đây là cách debug hiệu suất SQL:

```python
import logging
from rest_framework import viewsets
from django.db import connection, reset_queries
from .models import Product
from .serializers import ProductSerializer

logger = logging.getLogger(__name__)

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.select_related('category').all()  # Tránh N+1
    serializer_class = ProductSerializer

    def list(self, request, *args, **kwargs):
        reset_queries()  # Reset truy vấn để đo
        queryset = self.get_queryset()
        serializer = self.get_serializer(queryset, many=True)
        logger.info(f"SQL queries: {len(connection.queries)}")
        for query in connection.queries:
            logger.info(f"Query: {query['sql']}, Time: {query['time']}")
        return Response(serializer.data)
```

- **Debug**:
  - Gửi yêu cầu đến `/api/products/`.
  - Kiểm tra `debug.log` để xem số lượng truy vấn và thời gian:
    ```
    [2025-05-18 17:14:00] INFO: SQL queries: 1
    [2025-05-18 17:14:00] INFO: Query: SELECT ... FROM myapp_product LEFT OUTER JOIN myapp_category ..., Time: 0.002
    ```
  - Nếu số truy vấn >1, kiểm tra serializer hoặc queryset xem có truy cập trường liên quan không tối ưu không.
  - Dùng Debug Toolbar để xác nhận số truy vấn và thời gian tải trang.

- **Kiểm tra cache**:
  Nếu cache được sử dụng:
  ```python
  @cache_page(60 * 15)
  def list(self, request, *args, **kwargs):
      reset_queries()
      queryset = self.get_queryset()
      serializer = self.get_serializer(queryset, many=True)
      logger.info(f"SQL queries: {len(connection.queries)}")
      return Response(serializer.data)
  ```
  - Sau lần đầu tiên (cache miss), số truy vấn sẽ là 0 (cache hit) nếu cache Redis hoạt động đúng.

---

### **4. Mẹo debug hiệu suất SQL**
- **Tối ưu queryset**:
  - Luôn dùng `select_related` và `prefetch_related` khi cần.
  - Tránh `objects.all()` nếu chỉ cần một số cột:
    ```python
    Product.objects.values('name', 'price')
    ```
- **Kiểm tra index**:
  - Đảm bảo các cột lọc/sắp xếp có index trong mô hình.
- **Giảm truy vấn trùng lặp**:
  - Dùng Debug Toolbar để phát hiện truy vấn lặp và hợp nhất chúng.
- **Tận dụng cache**:
  - Kết hợp Redis cache (như trong cấu hình của bạn) để giảm truy vấn SQL cho dữ liệu ít thay đổi.
- **Kiểm tra production**:
  - Sử dụng công cụ như New Relic hoặc slow query log để debug trong môi trường thực tế.

---

### **5. Kết luận**
- **Công cụ chính**: Django Debug Toolbar, `connection.queries`, `EXPLAIN`, và Redis CLI (để kiểm tra cache).
- **Kỹ thuật chính**: Kiểm tra N+1, phân tích truy vấn chậm, tối ưu queryset, và xác nhận cache giảm truy vấn.
- **Áp dụng với ProductViewSet**: Thêm logging, dùng `select_related`, và kiểm tra cache hit/miss.

Nếu bạn cần ví dụ cụ thể hơn (ví dụ: debug một truy vấn chậm trong `ProductViewSet` hoặc cấu hình `django-silk`), hoặc muốn tôi phân tích một đoạn code cụ thể, hãy cho tôi biết!