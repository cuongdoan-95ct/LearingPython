### Tổng quan về "Caching in Django"

Cache giúp lưu trữ tạm thời dữ liệu để tránh lặp lại các tác vụ tốn tài nguyên như truy vấn database.

#### Nội dung chính:
1. **Setting up the cache** (Thiết lập cache)
2. **The per-site cache** (Cache toàn site)
3. **The per-view cache** (Cache từng view)
4. **Template fragment caching** (Cache đoạn template)
5. **The low-level cache API** (API cache cấp thấp)
6. **Downstream caches** (Cache hạ nguồn)
7. **Using Vary headers** (Sử dụng header Vary)

---

### 1. Setting up the cache (Thiết lập cache)

#### Tổng quan
- Đã được giải thích chi tiết trong câu hỏi trước, nhưng tôi sẽ tóm tắt lại và mở rộng cách sử dụng cụ thể.

#### Cấu hình ví dụ (Redis)
```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        },
        'TIMEOUT': 300,  # 5 phút
    }
}
```

#### Lưu và lấy dữ liệu
```python
from django.core.cache import cache

# Lưu dữ liệu
cache.set('user_info', {'id': 1, 'name': 'Nam'}, 60)  # 60 giây

# Lấy dữ liệu
user = cache.get('user_info')
print(user)  # {'id': 1, 'name': 'Nam'}

# Lấy hoặc tạo nếu không có
count = cache.get_or_set('visit_count', 0, 120)
cache.set('visit_count', count + 1)
print(count)  # 0 (lần đầu), 1 (lần sau)
```

---

### 2. The per-site cache (Cache toàn site)

#### Tổng quan
- Cache toàn bộ trang web bằng middleware, áp dụng cho mọi request.

---

#### ✅ Ưu điểm:
- Rất nhanh vì cache toàn bộ HTML của một request.
- Dễ thiết lập (chỉ cần middleware).
- Giảm đáng kể lượng xử lý của server với các trang tĩnh.

#### ❌ Nhược điểm:
- Mọi người dùng đều nhận cùng một bản HTML, **không phân biệt ai đang đăng nhập**.
- Không phù hợp cho các trang có nội dung cá nhân hoá (ví dụ: dashboard, profile...).

---

#### Cấu hình
- Thêm middleware:
```python
# settings.py
MIDDLEWARE = [
    'django.middleware.cache.UpdateCacheMiddleware',
    # Các middleware khác
    'django.middleware.common.CommonMiddleware',
    'django.middleware.cache.FetchFromCacheMiddleware',
]

CACHE_MIDDLEWARE_ALIAS = 'default'
CACHE_MIDDLEWARE_SECONDS = 600  # 10 phút
CACHE_MIDDLEWARE_KEY_PREFIX = 'site_cache'
```

#### Cách hoạt động
- `UpdateCacheMiddleware`: Lưu response vào cache.
- `FetchFromCacheMiddleware`: Lấy response từ cache nếu có.

#### Ví dụ
```python
# myapp/views.py
def blog_home(request):
    posts = Post.objects.all()
    return render(request, 'blog/home.html', {'posts': posts})


class ProductListView(ListView):
    """
    Product list view
    """

    template_name = "product/list.html"
    model = Product
    context_object_name = "product_list"
    cache_key = "product_list"

```
- Truy cập `/products`, response được cache 10 phút. Các request sau trả thẳng từ cache.

#### Lưu ý
- Chỉ cache response với mã trạng thái 200 và method GET.
- Không dùng nếu trang có nội dung động (như dữ liệu người dùng).

---

### 3. The per-view cache (Cache từng view)

#### Tổng quan
- Cache riêng cho từng view bằng decorator `cache_page`.


#### ✅ Ưu điểm:
- Có thể kiểm soát chính xác cache ở đâu.
- Không ảnh hưởng đến toàn bộ site.
- Có thể cá nhân hoá từng view bằng cách dùng `vary_on_cookie`, `vary_on_headers` hoặc custom key.

#### ❌ Nhược điểm:
- Không đơn giản như per-site nếu bạn cần phân biệt user.
- Không cache toàn bộ template, chỉ phần output của view.

---

#### 🔧 Cách dùng:

```python
from django.views.decorators.cache import cache_page

@cache_page(60 * 10)  # cache trong 10 phút
def product_list(request):
    products = Product.objects.all()
    return render(request, 'products/list.html', {'products': products})
```

---

## ✅ Ví dụ thực tế:

```python
@cache_page(60 * 15)
def top_news(request):
    news = News.objects.order_by('-views')[:5]
    return render(request, 'news/top.html', {'news': news})
```


#### Ví dụ cơ bản
```python
# myapp/views.py
from django.views.decorators.cache import cache_page
from django.http import HttpResponse

@cache_page(60)  # Cache 60 giây
def product_detail(request, product_id):
    return HttpResponse(f"Chi tiết sản phẩm {product_id}")
```
- Truy cập `/product/1`, response được cache 60 giây.

#### Cache với tham số động
```python
@cache_page(60, key_prefix='product_cache')
def dynamic_view(request, category):
    return HttpResponse(f"Danh mục: {category}")
```
- `key_prefix`: Tạo key cache riêng để tránh xung đột.

#### Sử dụng với URLconf
```python
# myapp/urls.py
from django.urls import path
from django.views.decorators.cache import cache_page
from myapp.views import product_detail

urlpatterns = [
    path('product/<int:product_id>/', cache_page(60)(product_detail), name='product_detail'),
]
```

---

### 4. Template fragment caching (Cache đoạn template)

#### Tổng quan
- Cache một phần của template thay vì toàn bộ trang.


#### ✅ Ưu điểm:
- Linh hoạt: chỉ cache phần cần thiết (sidebar, menu, widget…).
- Phù hợp cho layout có phần thay đổi, phần không.
- Giữ được tính cá nhân hoá cho các phần không được cache.

#### ❌ Nhược điểm:
- Chỉ cache phần template, không giảm truy vấn DB nếu không khéo xử lý.
- Cần cẩn thận khi dùng cùng logic trong view.

---

#### 🔧 Cách dùng:

```django
{% load cache %}

{% cache 600 latest_articles %}
  <ul>
    {% for article in articles %}
      <li>{{ article.title }}</li>
    {% endfor %}
  </ul>
{% endcache %}
```

#### Ví dụ
```html
<!-- myapp/templates/product_list.html -->
{% load cache %}

<h1>Danh sách sản phẩm</h1>
{% cache 300 product_list %}
    <ul>
    {% for product in products %}
        <li>{{ product.name }} - {{ product.price }}</li>
    {% endfor %}
    </ul>
{% endcache %}
```

#### Cách hoạt động
- `{% cache timeout key %}`: Lưu đoạn HTML trong 300 giây với key `product_list`.

#### Cache với biến
```html
{% cache 300 product_list user.id %}
    <p>Danh sách riêng cho user {{ user.id }}</p>
    <!-- Nội dung -->
{% endcache %}
```
- Key cache sẽ khác nhau cho mỗi `user.id`.

---


#### 📊 So sánh tổng quan

| Loại cache             | Phạm vi         | Ưu điểm                               | Nhược điểm                              | Dùng khi nào?                        |
|------------------------|------------------|----------------------------------------|------------------------------------------|--------------------------------------|
| **Per-site**           | Toàn bộ trang    | Cực nhanh, đơn giản                   | Không cá nhân hoá                        | Trang blog, tin tức, public pages    |
| **Per-view**           | Từng URL/view    | Kiểm soát tốt hơn, dễ hiểu            | Không phù hợp cho nội dung cá nhân hoá  | Trang danh sách, top sản phẩm        |
| **Fragment**           | Một phần template| Linh hoạt, giữ phần động               | Phức tạp nếu data vẫn query trong view  | Sidebar, danh mục, top 10, menu      |

---

### 5. The low-level cache API (API cache cấp thấp)

#### Tổng quan
- API cấp thấp cho phép kiểm soát chi tiết việc lưu/lấy dữ liệu từ cache.

#### Các phương thức
- **`set(key, value, timeout)`**: Lưu dữ liệu.
- **`get(key, default)`**: Lấy dữ liệu.
- **`add(key, value, timeout)`**: Lưu nếu key chưa tồn tại.
- **`get_or_set(key, default, timeout)`**: Lấy hoặc lưu.
- **`delete(key)`**: Xóa key.
- **`clear()`**: Xóa toàn bộ cache.

#### Ví dụ thực tế
```python
# myapp/views.py
from django.core.cache import cache
from django.http import HttpResponse
from myapp.models import Product

def product_api(request):
    cache_key = 'all_products'
    
    # Lấy từ cache
    products = cache.get(cache_key)
    if products is None:
        # Truy vấn DB nếu không có
        products = list(Product.objects.values('name', 'price'))
        cache.set(cache_key, products, 600)  # Lưu 10 phút
        return HttpResponse(f"Tạo mới: {len(products)} sản phẩm")
    
    # Trả từ cache
    return HttpResponse(f"Cache: {len(products)} sản phẩm")

def update_product(request, product_id):
    # Cập nhật DB
    product = Product.objects.get(id=product_id)
    product.price += 10
    product.save()
    
    # Xóa cache để làm mới
    cache.delete('all_products')
    return HttpResponse("Đã cập nhật và xóa cache")
```

#### Dùng nhiều cache
```python
# settings.py
CACHES = {
    'default': {'BACKEND': 'django.core.cache.backends.locmem.LocMemCache'},
    'redis': {'BACKEND': 'django_redis.cache.RedisCache', 'LOCATION': 'redis://127.0.0.1:6379/1'}
}

# myapp/views.py
from django.core.cache import caches

def multi_cache(request):
    redis_cache = caches['redis']
    redis_cache.set('redis_data', 'Dữ liệu từ Redis', 60)
    print(redis_cache.get('redis_data'))  # Dữ liệu từ Redis
```

---

### 6. Downstream caches (Cache hạ nguồn)

#### Tổng quan
- Cache ngoài Django (như proxy, CDN) dựa trên header HTTP.

#### Cấu hình
- Dùng `cache_control`:
```python
from django.views.decorators.cache import cache_control

@cache_control(max_age=3600)  # Cache 1 giờ ở client/proxy
def public_view(request):
    return HttpResponse("Trang công khai")
```

#### Ví dụ với điều kiện
```python
@cache_control(max_age=0, no_cache=True)  # Không cache
def private_view(request):
    return HttpResponse("Trang cá nhân")
```

---

### 7. Using Vary headers (Sử dụng header Vary)

#### Tổng quan
- Header `Vary` chỉ định điều kiện cache dựa trên request header (như `Accept-Language`).

#### Ví dụ
```python
from django.views.decorators.vary import vary_on_headers

@vary_on_headers('Accept-Language')
@cache_page(600)
def localized_view(request):
    lang = request.headers.get('Accept-Language', 'en')
    return HttpResponse(f"Ngôn ngữ: {lang}")
```
- Cache riêng cho từng ngôn ngữ.

#### Với cookie
```python
@vary_on_headers('Cookie')
@cache_page(300)
def user_specific_view(request):
    user_id = request.COOKIES.get('user_id', 'guest')
    return HttpResponse(f"User: {user_id}")
```

---

### Ví dụ tổng hợp

#### Model và Factory
```python
# myapp/models.py
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.IntegerField()

# myapp/factories.py
import factory
from factory.django import DjangoModelFactory
from myapp.models import Product

class ProductFactory(DjangoModelFactory):
    class Meta:
        model = Product
    name = factory.Sequence(lambda n: f"Product_{n}")
    price = factory.Faker('random_int', min=10, max=1000)
```

#### View với Cache
```python
# myapp/views.py
from django.core.cache import cache
from django.http import HttpResponse
from django.views.decorators.cache import cache_page
from myapp.models import Product

def product_list(request):
    cache_key = 'product_list'
    products = cache.get(cache_key)
    
    if products is None:
        products = list(Product.objects.values('name', 'price'))
        cache.set(cache_key, products, 300)  # 5 phút
        return HttpResponse(f"Tạo mới: {len(products)} sản phẩm")
    
    return HttpResponse(f"Cache: {len(products)} sản phẩm")

@cache_page(60)
def cached_product(request, product_id):
    product = Product.objects.get(id=product_id)
    return HttpResponse(f"{product.name}: ${product.price}")
```

#### Template với Cache
```html
<!-- myapp/templates/product_list.html -->
{% load cache %}
<h1>Sản phẩm</h1>
{% cache 300 product_list %}
    <ul>
    {% for product in products %}
        <li>{{ product.name }} - {{ product.price }}</li>
    {% endfor %}
    </ul>
{% endcache %}
```

#### URL
```python
# myapp/urls.py
from django.urls import path
from myapp.views import product_list, cached_product

urlpatterns = [
    path('', product_list, name='product_list'),
    path('product/<int:product_id>/', cached_product, name='cached_product'),
]
```

#### Tạo dữ liệu và chạy
```bash
python manage.py migrate
python manage.py shell
```
```python
from myapp.factories import ProductFactory
ProductFactory.create_batch(5)
exit()
python manage.py runserver
```
---

### Tóm tắt chi tiết

- **Thiết lập**: Cấu hình `CACHES` với backend (Redis, Memcached, v.v.).
- **Toàn site**: Middleware cache toàn bộ trang.
- **Từng view**: `@cache_page` cho view cụ thể.
- **Template**: `{% cache %}` cho đoạn template.
- **Cấp thấp**: `cache.set()`, `cache.get()` linh hoạt.
- **Hạ nguồn**: `cache_control` cho proxy/CDN.
- **Vary**: Cache theo header (ngôn ngữ, cookie).