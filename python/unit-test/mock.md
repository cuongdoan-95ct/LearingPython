### Tổng quan về `unittest.mock`

**`unittest.mock`** là một module trong Python giúp tạo các đối tượng giả (mock objects) để thay thế các thành phần thực trong kiểm thử. Nó rất hữu ích khi bạn muốn kiểm tra một phần mã mà không phụ thuộc vào các hệ thống bên ngoài (như API, database).

#### Nội dung chính:
1. **Giới thiệu về Mock**
2. **Cài đặt và import**
3. **Mock cơ bản (Mock class)**
4. **MagicMock**
5. **Patch (Vá - Mocking các đối tượng cụ thể)**
6. **Các thuộc tính và phương thức của Mock**
7. **Ví dụ thực tế**

---

### 1. Giới thiệu về Mock

#### Mock là gì?
- **Mock** là một đối tượng giả lập, thay thế cho các đối tượng thực trong mã nguồn.
- Dùng để:
  - Giả lập hành vi của hàm, lớp, hoặc module.
  - Kiểm tra tương tác (gọi hàm, truyền tham số) mà không chạy mã thực tế.

#### Tại sao dùng Mock?
- Tránh phụ thuộc vào hệ thống bên ngoài (database, API).
- Kiểm tra logic riêng lẻ.
- Giả lập lỗi hoặc tình huống đặc biệt.

---

### 2. Cài đặt và import

#### Không cần cài đặt
- `unittest.mock` có sẵn trong Python 3.3+. Chỉ cần import:
```python
from unittest.mock import Mock, MagicMock, patch
```

#### Kiểm tra
```python
import unittest.mock
print(unittest.mock.__version__)  # Không có thuộc tính này, nhưng import thành công là OK
```

---

### 3. Mock cơ bản (Mock class)

#### Tạo Mock
```python
from unittest.mock import Mock

# Tạo một mock
mock = Mock()

# Gọi mock như hàm
mock()  # Không làm gì, nhưng có thể kiểm tra

# Gán giá trị trả về
mock.return_value = 42
print(mock())  # 42
```

#### Kiểm tra lần gọi
```python
mock(1, 2, key='value')
mock.assert_called_once_with(1, 2, key='value')  # Kiểm tra gọi đúng tham số
```

#### Thuộc tính giả lập
```python
mock.name = "Nam"
print(mock.name)  # Nam
mock.name.upper.return_value = "NAM"
print(mock.name.upper())  # NAM
```

---

### 4. MagicMock

#### MagicMock là gì?
- `MagicMock` là phiên bản nâng cao của `Mock`, hỗ trợ các **magic method** (như `__len__`, `__str__`).

#### Ví dụ
```python
from unittest.mock import MagicMock

mock = MagicMock()
mock.__str__.return_value = "Xin chào"
print(str(mock))  # Xin chào

mock.__len__.return_value = 5
print(len(mock))  # 5
```

#### So sánh với Mock
- `Mock` không hỗ trợ magic method mặc định:
```python
mock = Mock()
len(mock)  # Lỗi: TypeError
```

---

### 5. Patch (Vá - Mocking các đối tượng cụ thể)

#### Patch là gì?
- `patch` thay thế một đối tượng thực bằng mock trong phạm vi cụ thể (thường dùng làm decorator hoặc context manager).

#### Ví dụ cơ bản (Decorator)
```python
from unittest.mock import patch

# Giả lập hàm print
@patch('builtins.print')
def test_function(mock_print):
    print("Hello")
    mock_print.assert_called_once_with("Hello")

test_function()
```

#### Với Context Manager
```python
with patch('builtins.print') as mock_print:
    print("Xin chào")
    mock_print.assert_called_once_with("Xin chào")
```

#### Patch module
```python
# mymodule.py
def get_data():
    return "Real data"

# test.py
from unittest.mock import patch
from mymodule import get_data

with patch('mymodule.get_data', return_value="Fake data") as mock_get:
    print(get_data())  # Fake data
    mock_get.assert_called_once()
```

#### Patch nhiều đối tượng
```python
with patch.multiple('mymodule', get_data=Mock(return_value="Mocked"), autospec=True):
    print(get_data())  # Mocked
```

---

### 6. Các thuộc tính và phương thức của Mock

#### a. Thuộc tính
- **`return_value`**: Giá trị trả về khi gọi mock:
```python
mock = Mock(return_value=10)
print(mock())  # 10
```
- **`side_effect`**: Giả lập nhiều giá trị hoặc lỗi:
```python
mock = Mock(side_effect=[1, 2, 3])
print(mock())  # 1
print(mock())  # 2
print(mock())  # 3

mock.side_effect = ValueError("Lỗi")
mock()  # Ném ValueError
```
- **`call_args`**: Tham số của lần gọi cuối:
```python
mock(1, key='value')
print(mock.call_args)  # call(1, key='value')
```
- **`call_count`**: Số lần gọi:
```python
mock(1)
mock(2)
print(mock.call_count)  # 2
```

#### b. Phương thức kiểm tra
- **`assert_called()`**: Kiểm tra mock đã được gọi:
```python
mock(1)
mock.assert_called()
```
- **`assert_called_once()`**: Gọi đúng 1 lần:
```python
mock(1)
mock.assert_called_once()
```
- **`assert_called_with()`**: Kiểm tra tham số:
```python
mock(1, 2)
mock.assert_called_with(1, 2)
```
- **`assert_not_called()`**: Chưa được gọi:
```python
mock.assert_not_called()
```

#### c. Reset Mock
```python
mock(1)
mock.reset_mock()
mock.assert_not_called()  # OK
```

---

### 7. Ví dụ thực tế

#### Với Django
Giả sử bạn có view gửi email:
```python
# myapp/views.py
from django.core.mail import send_mail

def send_welcome_email(user_email):
    send_mail("Chào mừng", "Xin chào!", "from@example.com", [user_email])
```

##### Kiểm thử với Mock
```python
# myapp/tests.py
from django.test import TestCase
from unittest.mock import patch
from myapp.views import send_welcome_email

class TestEmail(TestCase):
    @patch('django.core.mail.send_mail')
    def test_send_email(self, mock_send_mail):
        send_welcome_email("test@example.com")
        mock_send_mail.assert_called_once_with(
            "Chào mừng", "Xin chào!", "from@example.com", ["test@example.com"]
        )
```

#### Giả lập API bên ngoài
```python
# myapp/api.py
import requests

def fetch_data():
    response = requests.get("https://api.example.com/data")
    return response.json()
```

##### Kiểm thử
```python
from unittest.mock import Mock, patch
import myapp.api

def test_fetch_data():
    mock_response = Mock()
    mock_response.json.return_value = {"key": "value"}

    with patch('requests.get', return_value=mock_response):
        result = myapp.api.fetch_data()
        assert result == {"key": "value"}
        myapp.api.requests.get.assert_called_once_with("https://api.example.com/data")

test_fetch_data()
```

#### Giả lập lỗi
```python
with patch('requests.get', side_effect=requests.RequestException("Lỗi mạng")):
    try:
        myapp.api.fetch_data()
    except requests.RequestException as e:
        print(str(e))  # Lỗi mạng
```

---

### Tóm tắt

- **Mock**: Đối tượng giả cơ bản (`Mock`), hỗ trợ magic method (`MagicMock`).
- **Patch**: Thay thế đối tượng thực (decorator, context manager).
- **Thuộc tính**:
  - `return_value`, `side_effect`: Điều khiển đầu ra.
  - `call_args`, `call_count`: Theo dõi lần gọi.
- **Kiểm tra**: `assert_called`, `assert_called_with`, v.v.
- **Ứng dụng**: Giả lập API, database, email trong kiểm thử.

#### Ví dụ tổng hợp
```python
from unittest.mock import Mock, patch

# Mock cơ bản
mock = Mock(return_value=100)
print(mock())  # 100
mock.assert_called()

# MagicMock
magic = MagicMock()
magic.__add__.return_value = 5
print(2 + magic)  # 5

# Patch
with patch('builtins.input', return_value="Test"):
    print(input())  # Test
```