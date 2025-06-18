## Chương 3: Models

Chương này tập trung vào cách thiết kế và sử dụng **models** trong Django - nơi định nghĩa cấu trúc dữ liệu của ứng dụng. Nó cũng giới thiệu các mẫu thiết kế (patterns) để tối ưu hóa models.

### 1. M lớn hơn V và C ($M$ is bigger than $V$ and $C$)
#### Giải thích:
- Trong kiến trúc MTV (Model-Template-View) của Django, **Model** là phần quan trọng nhất vì nó định nghĩa dữ liệu - trái tim của ứng dụng.
- View và Template chỉ là cách hiển thị và xử lý dữ liệu từ Model.

### 2. Săn lùng Model (The model hunt)
#### Giải thích:
- Để thiết kế model, bạn cần xác định các thực thể (entities) trong ứng dụng và mối quan hệ giữa chúng.
- Dùng sơ đồ **Entity-Relationship (ER)** để hình dung.

#### Ví dụ:
- Trong SuperBook:
  - Thực thể: `SuperHero` (Siêu anh hùng), `Post` (Bài viết).
  - Quan hệ: Một SuperHero có thể đăng nhiều Post (`One-to-Many`).
```python
# posts/models.py
from django.db import models

class SuperHero(models.Model):
    name = models.CharField(max_length=100)
    power = models.CharField(max_length=100)

    def __str__(self):
        return self.name

class Post(models.Model):
    hero = models.ForeignKey(SuperHero, on_delete=models.CASCADE)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.hero.name}: {self.content[:20]}"
```

### 3. Chia file models.py thành nhiều file (Splitting models.py into multiple files)
#### Giải thích:
- Khi dự án lớn, file `models.py` có thể trở nên dài và khó quản lý. Bạn nên chia thành nhiều file nhỏ.

#### Ví dụ:
- Thay vì một file `models.py`:
```
posts/
├── models/
│   ├── __init__.py
│   ├── superhero.py  # Chứa SuperHero model
│   └── post.py       # Chứa Post model
```
- `superhero.py`:
```python
from django.db import models

class SuperHero(models.Model):
    name = models.CharField(max_length=100)
    power = models.CharField(max_length=100)

    def __str__(self):
        return self.name
```
- `post.py`:
```python
from django.db import models
from .superhero import SuperHero

class Post(models.Model):
    hero = models.ForeignKey(SuperHero, on_delete=models.CASCADE)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.hero.name}: {self.content[:20]}"
```
- `__init__.py`:
```python
from .superhero import SuperHero
from .post import Post
```

### 4. Các mẫu cấu trúc (Structural Patterns)

#### 4.1. Normalized Models (Mô hình chuẩn hóa)
##### Giải thích:
- Chuẩn hóa giúp giảm dư thừa dữ liệu (data redundancy) và đảm bảo tính nhất quán.
- Có 3 dạng chuẩn hóa chính: 1NF, 2NF, 3NF.

##### Ví dụ:
- Ban đầu, dữ liệu không chuẩn hóa:
```python
class Post(models.Model):
    hero_name = models.CharField(max_length=100)  # Tên siêu anh hùng
    hero_power = models.CharField(max_length=100) # Sức mạnh
    content = models.TextField()
```
- Chuẩn hóa (1NF - không lặp dữ liệu):
```python
class SuperHero(models.Model):
    name = models.CharField(max_length=100)
    power = models.CharField(max_length=100)

class Post(models.Model):
    hero = models.ForeignKey(SuperHero, on_delete=models.CASCADE)
    content = models.TextField()
```

#### 4.2. Model Mixins
##### Giải thích:
- Mixins là cách tái sử dụng code bằng cách thêm chức năng chung vào nhiều model.

##### Ví dụ:
- Thêm thời gian tạo và cập nhật vào model:
```python
# posts/models.py
class TimeStampedMixin(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True  # Không tạo bảng cho mixin

class SuperHero(TimeStampedMixin):
    name = models.CharField(max_length=100)
    power = models.CharField(max_length=100)

class Post(TimeStampedMixin):
    hero = models.ForeignKey(SuperHero, on_delete=models.CASCADE)
    content = models.TextField()
```

#### 4.3. User Profiles (Hồ sơ người dùng)
##### Giải thích:
- Tạo hồ sơ người dùng tùy chỉnh gắn với `User` của Django để thêm thông tin bổ sung.

##### Ví dụ:
```python
# accounts/models.py
from django.contrib.auth.models import User
from django.db import models

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    hero_name = models.CharField(max_length=100)
    power = models.CharField(max_length=100)

    def __str__(self):
        return self.hero_name
```

#### 4.4. Service Objects (Đối tượng dịch vụ)
##### Giải thích:
- Tách logic phức tạp khỏi model để giữ model đơn giản, dễ bảo trì.

##### Ví dụ:
- Tạo bài viết với kiểm tra quyền:
```python
# posts/services.py
from .models import Post, SuperHero

class PostService:
    @staticmethod
    def create_post(hero, content):
        if hero.power != "None":  # Kiểm tra quyền
            return Post.objects.create(hero=hero, content=content)
        raise ValueError("Chỉ siêu anh hùng có sức mạnh mới được đăng bài!")
```
- Sử dụng trong view:
```python
# posts/views.py
from django.shortcuts import render
from .services import PostService

def create_post(request):
    hero = SuperHero.objects.get(name="Captain Obvious")
    content = "Đánh bại robot khổng lồ!"
    post = PostService.create_post(hero, content)
    return render(request, "posts/success.html")
```

### 5. Các mẫu truy xuất (Retrieval Patterns)

#### 5.1. Property Field
##### Giải thích:
- Dùng `@property` để tính toán giá trị thay vì lưu vào cơ sở dữ liệu.

##### Ví dụ:
```python
class SuperHero(models.Model):
    name = models.CharField(max_length=100)
    power = models.CharField(max_length=100)

    @property
    def post_count(self):
        return self.post_set.count()  # Đếm số bài viết
```

#### 5.2. Custom Model Managers
##### Giải thích:
- Tùy chỉnh cách truy vấn dữ liệu bằng manager.

##### Ví dụ:
```python
class ActivePostManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(content__icontains="active")

class Post(models.Model):
    hero = models.ForeignKey(SuperHero, on_delete=models.CASCADE)
    content = models.TextField()
    objects = models.Manager()  # Manager mặc định
    active_objects = ActivePostManager()  # Manager tùy chỉnh

# Sử dụng
active_posts = Post.active_objects.all()  # Chỉ lấy bài có từ "active"
```

### 6. Migrations
#### Giải thích:
- Migration giúp cập nhật cấu trúc cơ sở dữ liệu khi model thay đổi.

#### Ví dụ:
- Sau khi thêm trường mới vào `Post`:
```python
class Post(models.Model):
    hero = models.ForeignKey(SuperHero, on_delete=models.CASCADE)
    content = models.TextField()
    likes = models.IntegerField(default=0)  # Trường mới
```
- Chạy lệnh:
```bash
python manage.py makemigrations
python manage.py migrate
```

---

### Tóm tắt
- **Chương 3**: Tập trung vào cách thiết kế model hiệu quả với các pattern như chuẩn hóa, mixins, và tối ưu truy xuất dữ liệu.

Bạn có muốn tôi giải thích thêm phần nào hoặc cung cấp ví dụ khác không?