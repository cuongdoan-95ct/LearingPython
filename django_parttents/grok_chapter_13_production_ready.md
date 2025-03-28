### Chương 13: Sẵn sàng triển khai sản phẩm (Tổng quan)

Chương này hướng dẫn cách đưa ứng dụng Django từ giai đoạn phát triển sang triển khai thực tế (production). Nó bao gồm việc chọn hạ tầng, triển khai tự động, giám sát và tối ưu hiệu suất để ứng dụng ổn định, đáng tin cậy và nhanh chóng.

#### Nội dung chính:
1. **Môi trường sản phẩm (Production Environment)**
2. **Công cụ triển khai (Deployment Tools)**
3. **Giám sát (Monitoring)**
4. **Cải thiện hiệu suất (Improving Performance)**

---

### 1. Môi trường sản phẩm (Production Environment)

#### Môi trường sản phẩm là gì?
Đây là giai đoạn ứng dụng được triển khai để người dùng thực tế sử dụng, đòi hỏi sự ổn định và bảo mật cao hơn môi trường phát triển.

#### Chọn một web stack
- **Web stack**: Tập hợp công nghệ (web server, ứng dụng server, database) để chạy ứng dụng.
- **Thành phần**:
  - **Web server**: Nginx (chuyển tiếp yêu cầu).
  - **Application server**: Gunicorn (chạy mã Python).
  - **Database**: PostgreSQL (lưu trữ dữ liệu).
- **Ví dụ cấu hình**:
```nginx
# nginx.conf
server {
    listen 80;
    server_name example.com;
    location / {
        proxy_pass http://127.0.0.1:8000;  # Gunicorn
    }
}
```
```bash
# Chạy Gunicorn
gunicorn myapp.wsgi:application --bind 127.0.0.1:8000
```

#### Virtual Machines hoặc Docker
- **Virtual Machines (VM)**: Máy ảo chạy toàn bộ hệ điều hành (như Ubuntu).
- **Docker**: Container nhẹ, chỉ chạy ứng dụng và phụ thuộc.
- **Ví dụ Docker**:
```Dockerfile
# Dockerfile
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["gunicorn", "myapp.wsgi:application", "--bind", "0.0.0.0:8000"]
```
```bash
# Xây dựng và chạy
docker build -t myapp .
docker run -p 8000:8000 myapp
```

#### Hosting
- **Platform as a Service (PaaS)**: Heroku, AWS Elastic Beanstalk – dễ triển khai.
- **Virtual Private Server (VPS)**: DigitalOcean, Linode – linh hoạt hơn.
- **Serverless**: AWS Lambda – chỉ trả phí khi sử dụng.
- **Ví dụ Heroku**:
```bash
heroku create myapp
git push heroku main
```

---

### 2. Công cụ triển khai (Deployment Tools)

#### Công cụ tự động
- **Fabric**: Tự động hóa triển khai qua SSH.
- **Ví dụ Fabric**:
```python
# fabfile.py
from fabric import task

@task
def deploy(c):
    c.run("git pull origin main")
    c.run("pip install -r requirements.txt")
    c.run("python manage.py migrate")
    c.run("systemctl restart gunicorn")
```
```bash
fab -H user@server deploy
```

#### Các bước triển khai điển hình
1. Kéo mã từ Git.
2. Cài đặt phụ thuộc.
3. Chạy migrations.
4. Khởi động lại server.

#### Quản lý cấu hình
- Dùng Ansible hoặc Chef để đồng bộ cấu hình server.

---

### 3. Giám sát (Monitoring)

#### Giám sát là gì?
Theo dõi hiệu suất và lỗi để phản ứng kịp thời.

#### Công cụ
- **Graphite**: Theo dõi số liệu (CPU, request rate).
- **Sentry**: Ghi lại lỗi ứng dụng.
- **Ví dụ Sentry**:
```python
# settings.py
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration

sentry_sdk.init(
    dsn="https://your-dsn@sentry.io/123",
    integrations=[DjangoIntegration()],
    traces_sample_rate=1.0
)
```
- Lỗi sẽ được gửi đến Sentry để phân tích.

---

### 4. Cải thiện hiệu suất (Improving Performance)

#### Hiệu suất frontend
- **Cache vô hạn**: Lưu dữ liệu tĩnh mãi mãi, cập nhật bất đồng bộ.
- **Ví dụ cache với Redis**:
```python
# settings.py
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",
    }
}
```
```python
# views.py
from django.core.cache import cache

def trang_chu(request):
    data = cache.get("trang_chu")
    if not data:
        data = SanPham.objects.all()
        cache.set("trang_chu", data, timeout=None)  # Cache mãi mãi
    return render(request, "trang_chu.html", {"data": data})
```
- **Static asset manager**: Nén CSS/JS với `django-pipeline`.
```python
# settings.py
INSTALLED_APPS += ["pipeline"]
STATICFILES_STORAGE = "pipeline.storage.PipelineStorage"
PIPELINE = {
    "CSS_COMPRESSOR": "pipeline.compressors.yui.YUICompressor",
}
```

#### Hiệu suất backend
- **Template**: Dùng cached template loader.
```python
# settings.py
TEMPLATES = [{
    "BACKEND": "django.template.backends.django.DjangoTemplates",
    "OPTIONS": {
        "loaders": [
            ("django.template.loaders.cached.Loader", [
                "django.template.loaders.filesystem.Loader",
                "django.template.loaders.app_directories.Loader",
            ]),
        ],
    },
}]
```
- **Database**:
  - `select_related`: Giảm truy vấn liên quan.
```python
# views.py
def danh_sach_san_pham(request):
    san_pham = SanPham.objects.select_related("danh_muc").all()
    return render(request, "danh_sach.html", {"san_pham": san_pham})
```
  - `bulk_create`: Tạo nhiều bản ghi cùng lúc.
```python
SanPham.objects.bulk_create([
    SanPham(ten="iPhone", gia=1000),
    SanPham(ten="Samsung", gia=800),
])
```
- **Caching**:
  - **Cached session**: Lưu session trong Redis.
```python
SESSION_ENGINE = "django.contrib.sessions.backends.cache"
SESSION_CACHE_ALIAS = "default"
```
  - **Caching frameworks**: Dùng `django-cachalot` để cache truy vấn ORM.
```bash
pip install django-cachalot
```
```python
# settings.py
INSTALLED_APPS += ["cachalot"]
```
  - **Caching patterns**: "Russian doll caching" – cache từng phần template.

#### Ví dụ thực tế:
- Trang sản phẩm tải chậm → Thêm cache Redis, nén CSS/JS, dùng `select_related` → Tăng tốc 10x.

---

### Tóm tắt
- **Môi trường sản phẩm**: Chọn stack (Nginx, Gunicorn, PostgreSQL), dùng Docker hoặc PaaS.
- **Công cụ triển khai**: Fabric tự động hóa, quản lý cấu hình.
- **Giám sát**: Graphite (hiệu suất), Sentry (lỗi).
- **Hiệu suất**:
  - **Frontend**: Cache vô hạn, nén static files.
  - **Backend**: Cache template, tối ưu database, dùng Redis.