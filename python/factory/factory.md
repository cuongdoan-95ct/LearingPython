# Tổng quan về Factory Boy

**Factory Boy** là một thư viện giúp tạo dữ liệu mẫu nhanh chóng và linh hoạt cho các bài kiểm thử. Nó thay thế việc tạo dữ liệu thủ công hoặc dùng fixtures tĩnh (như JSON) bằng cách cung cấp các "factory" (nhà máy) để sinh dữ liệu động.

#### Nội dung chính:
1. **Giới thiệu (Introduction)**
2. **Cài đặt (Installation)**
3. **Cách sử dụng cơ bản (Getting Started)**
4. **Các tính năng chính (Key Features)**
5. **Tích hợp với Django (Django Integration)**

---

### 1. Giới thiệu (Introduction)

#### Factory Boy là gì?
- Factory Boy giúp tạo đối tượng mẫu (objects) cho kiểm thử mà không cần viết dữ liệu thủ công lặp đi lặp lại.
- Hữu ích trong:
  - Kiểm thử đơn vị (unit testing).
  - Tạo dữ liệu giả lập cho phát triển.

#### Tại sao dùng Factory Boy?
- Dễ dàng tạo dữ liệu phức tạp.
- Tự động hóa việc tạo dữ liệu quan he (related objects).
- Linh hoạt hơn fixtures tĩnh.

---

### 2. Cài đặt (Installation)

#### Cài đặt qua pip
```bash
pip install factory_boy
```
- Nếu dùng với Django:
```bash
pip install factory_boy
```
- Yêu cầu: Python 3.6+.

#### Kiểm tra cài đặt
- Chạy Python shell:
```python
import factory
print(factory.__version__)  # Ví dụ: 3.3.0
```

---

### 3. Getting Started

#### Tạo Factory cơ bản
- Factory là một lớp kế thừa từ `factory.Factory`, định nghĩa cách tạo đối tượng.

##### Ví dụ cơ bản (Không dùng ORM)
```python
# factories.py
import factory

class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

class UserFactory(factory.Factory):
    class Meta:
        model = User

    name = "Nam"
    age = 25

# Sử dụng
user = UserFactory()
print(user.name, user.age)  # Nam 25
```

#### Tạo nhiều đối tượng
```python
users = UserFactory.create_batch(3)
for u in users:
    print(u.name, u.age)  # Nam 25 (3 lần)
```

#### Ghi đè giá trị
```python
user = UserFactory(name="Lan", age=30)
print(user.name, user.age)  # Lan 30
```

---

### 4. Các tính năng chính (Key Features)

#### a. Lazy Attributes (Thuộc tính lười biếng)
LazyAttribute được dùng để tạo giá trị động cho một field trong factory, dựa trên các field khác của object đó.
LazyFunction(...): Dùng khi bạn chỉ cần tạo giá trị ngẫu nhiên, không phụ thuộc field nào.
- Tạo giá trị động bằng hàm:
```python
class UserFactory(factory.Factory):
    class Meta:
        model = User

    name = factory.LazyFunction(lambda: "Người dùng " + str(factory.random.randgen.randint(1, 100)))
    age = factory.LazyAttribute(lambda obj: factory.random.randgen.randint(18, 60))

user = UserFactory()
print(user.name, user.age)  # Ví dụ: Người dùng 42 35
```

#### b. Sequence (Chuỗi)
- Tạo giá trị tăng dần:
```python
class UserFactory(factory.Factory):
    class Meta:
        model = User

    name = factory.Sequence(lambda n: f"User_{n}")
    age = 20

users = UserFactory.create_batch(3)
for u in users:
    print(u.name)  # User_0, User_1, User_2
```

#### c. Faker (Dữ liệu giả)
- Dùng `factory.Faker` để tạo dữ liệu thực tế:
```python
class UserFactory(factory.Factory):
    class Meta:
        model = User

    name = factory.Faker('name')  # Tên ngẫu nhiên
    age = factory.Faker('random_int', min=18, max=90)

user = UserFactory()
print(user.name, user.age)  # Ví dụ: John Doe 42
```

#### d. SubFactory (Nhà máy con)
- Tạo đối tượng liên quan:
```python
class Address:
    def __init__(self, city):
        self.city = city

class AddressFactory(factory.Factory):
    class Meta:
        model = Address
    city = "Hà Nội"

class UserFactory(factory.Factory):
    class Meta:
        model = User
    name = "Nam"
    address = factory.SubFactory(AddressFactory)

user = UserFactory()
print(user.name, user.address.city)  # Nam Hà Nội
```

#### e. Post-generation hooks (Xử lý sau tạo)
- Thực thi logic sau khi đối tượng được tạo:
```python
class UserFactory(factory.Factory):
    class Meta:
        model = User
    name = "Nam"
    age = factory.PostGenerationMethodCall('__init__', name="Nam", age=25)

user = UserFactory(age=30)  # Ghi đè age
print(user.age)  # 30
```

---

### 5. Tích hợp với Django (Django Integration)

#### Cài đặt với Django
- Factory Boy tích hợp tốt với Django qua `factory.django.DjangoModelFactory`.

#### Ví dụ với Django Model
Giả sử bạn có model:
```python
# myapp/models.py
from django.db import models

class User(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    age = models.IntegerField()

    def __str__(self):
        return self.name
```

##### Tạo Factory
```python
# myapp/factories.py
import factory
from factory.django import DjangoModelFactory
from myapp.models import User

class UserFactory(DjangoModelFactory):
    class Meta:
        model = User

    name = factory.Faker('name')
    email = factory.Sequence(lambda n: f"user{n}@example.com")
    age = factory.Faker('random_int', min=18, max=90)
```

##### Sử dụng trong kiểm thử
```python
# myapp/tests.py
from django.test import TestCase
from myapp.factories import UserFactory

class TestUser(TestCase):
    def test_create_user(self):
        user = UserFactory()
        self.assertTrue(User.objects.filter(email=user.email).exists())
        print(user.name, user.email)  # Ví dụ: Alice user0@example.com

    def test_batch_users(self):
        users = UserFactory.create_batch(3)
        self.assertEqual(User.objects.count(), 3)
```

#### Liên kết với model khác
Giả sử có model liên quan:
```python
# myapp/models.py
class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField()
```

##### Factory với SubFactory
```python
class ProfileFactory(DjangoModelFactory):
    class Meta:
        model = Profile
    user = factory.SubFactory(UserFactory)
    bio = factory.Faker('text')

# Sử dụng
profile = ProfileFactory()
print(profile.user.name, profile.bio)  # Ví dụ: Bob Một đoạn văn ngẫu nhiên
```

#### Tạo dữ liệu phức tạp
```python
class UserFactory(DjangoModelFactory):
    class Meta:
        model = User
    name = factory.Faker('name')
    email = factory.LazyAttribute(lambda o: f"{o.name.lower().replace(' ', '')}@example.com")
    age = 25

user = UserFactory()
print(user.email)  # Ví dụ: johnsmith@example.com
```

---

### Tóm tắt

- **Cài đặt**: `pip install factory_boy`.
- **Cơ bản**: Tạo factory với `factory.Factory` hoặc `DjangoModelFactory`.
- **Tính năng**:
  - `LazyAttribute`: Giá trị động.
  - `Sequence`: Giá trị tăng dần.
  - `Faker`: Dữ liệu giả thực tế.
  - `SubFactory`: Đối tượng liên quan.
  - `PostGeneration`: Xử lý sau tạo.
- **Django**: Dùng `DjangoModelFactory` để tạo model instance.

#### Ví dụ tổng hợp với Django
```python
# myapp/factories.py
import factory
from factory.django import DjangoModelFactory
from myapp.models import User, Profile

class UserFactory(DjangoModelFactory):
    class Meta:
        model = User
    name = factory.Faker('name')
    email = factory.Sequence(lambda n: f"user{n}@example.com")
    age = factory.Faker('random_int', min=18, max=60)

class ProfileFactory(DjangoModelFactory):
    class Meta:
        model = Profile
    user = factory.SubFactory(UserFactory)
    bio = factory.Faker('paragraph')

# myapp/tests.py
from django.test import TestCase
from myapp.factories import UserFactory, ProfileFactory

class TestModels(TestCase):
    def test_profile(self):
        profile = ProfileFactory()
        self.assertEqual(profile.user.profile, profile)
        print(profile.user.name, profile.bio)
```

#### Chạy kiểm thử
```bash
python manage.py test
```

---

### Ứng dụng thực tế
- **Kiểm thử**: Tạo dữ liệu mẫu nhanh chóng cho unit test.
- **Phát triển**: Tạo dữ liệu giả để thử giao diện hoặc API.