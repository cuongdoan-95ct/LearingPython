### Tổng quan về "Setting up the cache" trong Django

**Cache** trong Django là cơ chế lưu trữ tạm thời dữ liệu để giảm thời gian truy vấn, đặc biệt hữu ích cho các truy vấn database hoặc tính toán nặng. Phần này hướng dẫn cách cấu hình cache với các backend khác nhau.

#### Nội dung chính:
1. **Giới thiệu về Cache trong Django**
2. **Cấu hình Cache**
   - Memcached
   - Database caching
   - Filesystem caching
   - Local-memory caching
   - Dummy caching
3. **Cài đặt và sử dụng**

---

### 1. Giới thiệu về Cache trong Django

#### Cache là gì?
- Cache lưu trữ kết quả của các tác vụ tốn tài nguyên (như truy vấn database) để sử dụng lại thay vì tính toán lại.
- Django hỗ trợ nhiều backend cache như Memcached, Redis, database, file, hoặc bộ nhớ cục bộ.

#### Tại sao cần thiết lập Cache?
- Tăng tốc độ ứng dụng.
- Giảm tải cho database hoặc dịch vụ bên ngoài.

---

### 2. Cấu hình Cache

Django dùng biến `CACHES` trong `settings.py` để cấu hình cache. Mỗi backend có cách cài đặt riêng.

#### a. Memcached
- **Memcached** là hệ thống cache phân tán, nhanh và hiệu quả.
- Yêu cầu: Cài Memcached server và thư viện Python.

##### Cài đặt
- Cài Memcached trên máy:
  - Ubuntu: `sudo apt-get install memcached`
  - Mac: `brew install memcached`
- Cài thư viện Python:
```bash
pip install python-memcached  # hoặc pylibmc
```
- Khởi động Memcached:
```bash
memcached
```

##### Cấu hình
```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
        'LOCATION': '127.0.0.1:11211',  # Địa chỉ Memcached
    }
}
```

##### Ví dụ
```python
from django.core.cache import cache

cache.set('my_key', 'Xin chào', 30)  # Lưu 30 giây
print(cache.get('my_key'))  # Xin chào
```

#### b. Database Caching
- Lưu cache trong bảng database.

##### Cài đặt
- Tạo bảng cache:
```bash
python manage.py createcachetable
```
- Bảng `django_cache` sẽ được tạo trong database.

##### Cấu hình
```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'my_cache_table',  # Tên bảng
    }
}
```

##### Ví dụ
```python
cache.set('user_count', 100, 60)  # Lưu 60 giây
print(cache.get('user_count'))    # 100
```

#### c. Filesystem Caching
- Lưu cache dưới dạng file trên đĩa.

##### Cấu hình
```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': '/var/tmp/django_cache',  # Đường dẫn thư mục
    }
}
```
- Đảm bảo thư mục tồn tại và có quyền ghi:
```bash
mkdir -p /var/tmp/django_cache
chmod 777 /var/tmp/django_cache
```

##### Ví dụ
```python
cache.set('data', 'Hello World', 300)  # Lưu 5 phút
print(cache.get('data'))  # Hello World
```

#### d. Local-memory Caching
- Lưu cache trong RAM của tiến trình Python (nhanh nhưng không chia sẻ giữa các tiến trình).

##### Cấu hình
```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'unique-snowflake',  # Tên duy nhất cho cache
    }
}
```

##### Ví dụ
```python
cache.set('temp', 'Tạm thời', 10)  # Lưu 10 giây
print(cache.get('temp'))  # Tạm thời
```

#### e. Dummy Caching
- Không thực sự lưu cache, dùng để kiểm thử.

##### Cấu hình
```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.dummy.DummyCache',
    }
}
```

##### Ví dụ
```python
cache.set('fake', 'Không lưu', 100)
print(cache.get('fake'))  # None (không lưu)
```

#### f. Backend khác (Ví dụ: Redis)
- Dùng Redis làm cache (phổ biến trong thực tế).
- Cài đặt:
```bash
pip install django-redis
```
- Cấu hình:
```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',  # Redis server
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}
```
- Chạy Redis:
```bash
redis-server
```

##### Ví dụ
```python
cache.set('redis_key', 'Giá trị', 60)
print(cache.get('redis_key'))  # Giá trị
```

---

### 3. Cài đặt và sử dụng

#### Cấu hình nhiều Cache
- Có thể dùng nhiều backend cùng lúc:
```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
    },
    'memcached': {
        'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
        'LOCATION': '127.0.0.1:11211',
    }
}
```
- Chọn cache cụ thể:
```python
from django.core.cache import caches

mem_cache = caches['memcached']
mem_cache.set('key', 'Memcached đây', 30)
print(mem_cache.get('key'))  # Memcached đây
```

#### Thiết lập timeout
- Thời gian hết hạn (timeout) mặc định:
```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'TIMEOUT': 300,  # 5 phút
    }
}
```

#### Kiểm thử
```python
# myapp/views.py
from django.core.cache import cache
from django.http import HttpResponse

def test_cache(request):
    value = cache.get('test_key')
    if value is None:
        value = "Được tạo mới"
        cache.set('test_key', value, 60)  # Lưu 60 giây
    return HttpResponse(value)
```

---

### Tóm tắt

- **Backend**:
  - **Memcached**: Nhanh, phân tán (`PyMemcacheCache`).
  - **Database**: Lưu trong bảng (`DatabaseCache`).
  - **Filesystem**: Lưu file (`FileBasedCache`).
  - **Local-memory**: Nhanh, cục bộ (`LocMemCache`).
  - **Dummy**: Không lưu (`DummyCache`).
  - **Redis**: Phổ biến, mạnh mẽ (`django-redis`).
- **Cấu hình**: Dùng `CACHES` trong `settings.py`.
- **Sử dụng**: `cache.set()`, `cache.get()`.

#### Ví dụ tổng hợp
```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
    }
}

# myapp/views.py
from django.core.cache import cache
from django.http import HttpResponse

def home(request):
    if not cache.get('welcome'):
        cache.set('welcome', 'Chào mừng đến với Django', 30)
    return HttpResponse(cache.get('welcome'))
```

#### Chạy thử
```bash
python manage.py runserver
```
- Truy cập `http://localhost:8000`, thấy "Chào mừng đến với Django" trong 30 giây, sau đó tạo lại.

---

### Ứng dụng thực tế
- Cache kết quả truy vấn database.
- Lưu trữ dữ liệu tạm thời (như session, API response).

Hy vọng hướng dẫn này giúp bạn hiểu rõ cách thiết lập cache trong Django! Nếu bạn cần thêm ví dụ hoặc giải thích cụ thể (như dùng Redis với Celery), hãy cho tôi biết nhé!