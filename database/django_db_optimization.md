### Tổng quan về "Database access optimization" trong Django

Phần này cung cấp các kỹ thuật và chiến lược để giảm tải cơ sở dữ liệu, tăng tốc độ xử lý truy vấn, và tối ưu hóa hiệu suất ứng dụng Django.

#### Nội dung chính:
1. **Profile first** (Phân tích trước)
2. **Use standard database optimization techniques** (Sử dụng kỹ thuật tối ưu hóa cơ bản)
3. **Understand QuerySets** (Hiểu về QuerySets)
4. **Do database work in the database rather than in Python** (Thực hiện công việc trong DB thay vì Python)
5. **Retrieve everything at once if you know you will need it** (Lấy tất cả cùng lúc nếu cần)
6. **Don’t retrieve things you don’t need** (Không lấy dữ liệu không cần)
7. **Use bulk methods** (Sử dụng phương thức hàng loạt)
8. **Use query expressions and database functions** (Sử dụng biểu thức truy vấn và hàm DB)

---

### 1. Profile first (Phân tích trước)

#### Tổng quan
- Trước khi tối ưu, cần xác định bottleneck (điểm nghẽn) bằng cách phân tích hiệu suất.
- Dùng công cụ như Django Debug Toolbar hoặc logging để đo thời gian truy vấn.

#### Ví dụ
- Cài Django Debug Toolbar:
```bash
pip install django-debug-toolbar
```
```python
# settings.py
INSTALLED_APPS = [
    ...
    'debug_toolbar',
]
MIDDLEWARE = [
    ...
    'debug_toolbar.middleware.DebugToolbarMiddleware',
]
INTERNAL_IPS = ['127.0.0.1']
```
- Truy cập trang web, toolbar hiển thị số truy vấn và thời gian thực thi.

#### Lưu ý
- Chỉ tối ưu khi biết rõ vấn đề, tránh tối ưu hóa sớm (premature optimization).

---

### 2. Use standard database optimization techniques (Kỹ thuật tối ưu hóa cơ bản)

#### Tổng quan
- Áp dụng các kỹ thuật tối ưu DB truyền thống như:
  - Thêm **index** cho cột thường xuyên truy vấn.
  - Dùng **denormalization** (phi chuẩn hóa) nếu cần tốc độ.

#### Ví dụ: Thêm Index
```python
# myapp/models.py
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=100, db_index=True)  # Thêm index cho name
    price = models.IntegerField()

    class Meta:
        indexes = [
            models.Index(fields=['price']),  # Index cho price
        ]
```
- Chạy migration:
```bash
python manage.py makemigrations
python manage.py migrate
```

#### Lưu ý
- Index tăng tốc SELECT nhưng chậm INSERT/UPDATE, cân nhắc khi dùng.

---

### 3. Understand QuerySets (Hiểu về QuerySets)

#### Tổng quan
- **QuerySet** là lazy (lười biếng): Truy vấn chỉ thực thi khi được đánh giá (evaluated).
- Hiểu cách QuerySet hoạt động giúp tránh truy vấn dư thừa.

#### Ví dụ: Lazy Evaluation
```python
products = Product.objects.all()  # Không thực thi ngay
print(products)  # Lúc này mới truy vấn DB
```

#### Kiểm tra truy vấn
- Dùng `django.db.connection`:
```python
from django.db import connection

products = Product.objects.all()
print(len(connection.queries))  # 0 (chưa thực thi)
list(products)  # Thực thi
print(len(connection.queries))  # 1
```

#### Lưu ý
- Tránh lặp lại QuerySet trong vòng lặp, gây nhiều truy vấn.

---

### 4. Do database work in the database rather than in Python

#### Tổng quan
- Đẩy logic xử lý (filter, aggregate) sang DB thay vì Python để giảm tải.

#### Ví dụ: Filter trong DB
```python
# Không tối ưu: Lấy hết rồi lọc trong Python
products = Product.objects.all()
cheap_products = [p for p in products if p.price < 100]  # Truy vấn toàn bộ

# Tối ưu: Lọc trong DB
cheap_products = Product.objects.filter(price__lt=100)  # Chỉ truy vấn cần thiết
```

#### Ví dụ: Aggregate trong DB
```python
from django.db.models import Sum

# Không tối ưu: Tính tổng trong Python
total = sum(p.price for p in Product.objects.all())

# Tối ưu: Dùng DB
total = Product.objects.aggregate(Sum('price'))['price__sum']
```

#### Lưu ý
- DB thường nhanh hơn Python cho các phép toán lớn.

---

### 5. Retrieve everything at once if you know you will need it

#### Tổng quan
- Nếu biết trước cần toàn bộ dữ liệu liên quan, dùng `select_related` (cho ForeignKey) và `prefetch_related` (cho ManyToMany/Reverse ForeignKey) để giảm truy vấn.

#### Ví dụ: select_related
```python
# myapp/models.py
class Category(models.Model):
    name = models.CharField(max_length=100)

class Product(models.Model):
    name = models.CharField(max_length=100)
    category = models.ForeignKey(Category, on_delete=models.CASCADE)

# Không tối ưu: 1 + N truy vấn
products = Product.objects.all()
for p in products:
    print(p.category.name)  # Truy vấn riêng cho mỗi category

# Tối ưu: 1 truy vấn
products = Product.objects.select_related('category')
for p in products:
    print(p.category.name)  # Không truy vấn thêm
```

#### Ví dụ: prefetch_related
```python
# myapp/models.py
class Product(models.Model):
    name = models.CharField(max_length=100)
    tags = models.ManyToManyField('Tag')

class Tag(models.Model):
    name = models.CharField(max_length=100)

# Không tối ưu: 1 + N truy vấn
products = Product.objects.all()
for p in products:
    print(p.tags.all())  # Truy vấn riêng cho mỗi product

# Tối ưu: 2 truy vấn
products = Product.objects.prefetch_related('tags')
for p in products:
    print(p.tags.all())  # Không truy vấn thêm
```

#### Lưu ý
- `select_related`: Cho quan hệ 1-1 hoặc 1-nhiều.
- `prefetch_related`: Cho quan hệ nhiều-nhiều hoặc ngược.

---

### 6. Don’t retrieve things you don’t need

#### Tổng quan
- Chỉ lấy dữ liệu cần thiết bằng `values()`, `values_list()`, `only()`, hoặc `defer()` để giảm tải.

#### Ví dụ: values()
```python
# Không tối ưu: Lấy toàn bộ object
products = Product.objects.all()
names = [p.name for p in products]

# Tối ưu: Chỉ lấy name
names = Product.objects.values('name')  # Trả dict
```

#### Ví dụ: values_list()
```python
names = Product.objects.values_list('name', flat=True)  # Trả list
print(list(names))  # ['Product_1', 'Product_2', ...]
```

#### Ví dụ: only() và defer()
```python
# Chỉ lấy name
products = Product.objects.only('name')  # Tải name, bỏ qua price
for p in products:
    print(p.name)  # OK
    print(p.price)  # Truy vấn thêm

# Bỏ qua price
products = Product.objects.defer('price')  # Tải hết trừ price
```

#### Lưu ý
- `only()` và `defer()` vẫn tải ID, cẩn thận khi dùng.

---

### 7. Use bulk methods (Phương thức hàng loạt)

#### Tổng quan
- Dùng `bulk_create()`, `bulk_update()`, `delete()` để xử lý nhiều bản ghi cùng lúc, giảm truy vấn.

#### Ví dụ: bulk_create
```python
# Không tối ưu: Tạo từng bản ghi
for i in range(100):
    Product.objects.create(name=f"Product_{i}", price=i)

# Tối ưu: Tạo hàng loạt
products = [Product(name=f"Product_{i}", price=i) for i in range(100)]
Product.objects.bulk_create(products)
```

#### Ví dụ: bulk_update
```python
# Không tối ưu: Cập nhật từng bản ghi
products = Product.objects.all()
for p in products:
    p.price += 10
    p.save()

# Tối ưu: Cập nhật hàng loạt
products = Product.objects.all()
for p in products:
    p.price += 10
Product.objects.bulk_update(products, ['price'])
```

#### Lưu ý
- `bulk_create` không gọi signal, cẩn thận nếu dùng signal.

---

### 8. Use query expressions and database functions

#### Tổng quan
- Dùng `F()`, `Q()`, và các hàm DB (`Upper`, `Lower`, `Coalesce`, v.v.) để xử lý logic trong DB.

#### Ví dụ: F()
```python
# Không tối ưu: Lấy rồi cập nhật
for.Concurrent p in Product.objects.all():
    p.price = p.price * 2
    p.save()

# Tối ưu: Cập nhật trong DB
from django.db.models import F
Product.objects.update(price=F('price') * 2)
```

#### Ví dụ: Q()
```python
from django.db.models import Q

# Tìm sản phẩm giá < 100 hoặc tên chứa "Special"
products = Product.objects.filter(Q(price__lt=100) | Q(name__contains='Special'))
```

#### Ví dụ: Database Function
```python
from django.db.models.functions import Upper

# Chuyển tên thành chữ hoa trong DB
products = Product.objects.annotate(name_upper=Upper('name'))
for p in products:
    print(p.name_upper)  # PRODUCT_1
```

#### Lưu ý
- Kiểm tra hàm DB có hỗ trợ bởi backend (PostgreSQL, MySQL, v.v.) không.

---

### Ví dụ tổng hợp

#### Model
```python
# myapp/models.py
from django.db import models

class Category(models.Model):
    name = models.CharField(max_length=100)

class Product(models.Model):
    name = models.CharField(max_length=100, db_index=True)
    price = models.IntegerField()
    category = models.ForeignKey(Category, on_delete=models.CASCADE)
    tags = models.ManyToManyField('Tag')

class Tag(models.Model):
    name = models.CharField(max_length=100)
```

#### View tối ưu
```python
# myapp/views.py
from django.http import JsonResponse
from myapp.models import Product, Category
from django.db.models import F, Sum

def product_list(request):
    # Tối ưu: Lấy dữ liệu cần thiết, preload quan hệ
    products = Product.objects.select_related('category').prefetch_related('tags').filter(price__lt=1000)
    data = [
        {
            'name': p.name,
            'category': p.category.name,
            'tags': [t.name for t in p.tags.all()]
        } for p in products
    ]
    return JsonResponse({'products': data})

def update_prices(request):
    # Tăng giá tất cả sản phẩm thêm 10%
    Product.objects.update(price=F('price') * 1.1)
    total = Product.objects.aggregate(Sum('price'))['price__sum']
    return JsonResponse({'new_total': total})
```

#### Tạo dữ liệu
```bash
python manage.py shell
```
```python
from myapp.models import Category, Product, Tag
c = Category.objects.create(name="Electronics")
tags = [Tag.objects.create(name=f"Tag_{i}") for i in range(3)]
for i in range(10):
    p = Product.objects.create(name=f"Product_{i}", price=i * 100, category=c)
    p.tags.set(tags[:2])
exit()
```

---

### Tóm tắt chi tiết

- **Phân tích**: Dùng toolbar để tìm bottleneck.
- **Index**: Thêm index cho cột thường truy vấn.
- **QuerySet**: Lazy, chỉ thực thi khi cần.
- **DB logic**: Filter, aggregate trong DB.
- **Preload**: `select_related`, `prefetch_related`.
- **Chỉ lấy cần**: `values()`, `only()`, `defer()`.
- **Hàng loạt**: `bulk_create`, `bulk_update`.
- **Biểu thức**: `F()`, `Q()`, hàm DB.

Hy vọng hướng dẫn này giúp bạn hiểu rõ cách tối ưu hóa truy vấn trong Django! Nếu cần thêm ví dụ hoặc áp dụng cụ thể, hãy cho tôi biết nhé!