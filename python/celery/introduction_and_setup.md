### Tổng quan

​Celery là một thư viện mã nguồn mở trong Python, được thiết kế để quản lý và thực thi các tác vụ bất đồng bộ (asynchronous tasks) cũng như lập lịch công việc (task scheduling). Nó cho phép bạn tách biệt các công việc không cần thiết phải thực thi ngay lập tức ra khỏi luồng chính của ứng dụng, giúp cải thiện hiệu suất và trải nghiệm người dùng

#### Nội dung chính:
1. **Chọn một Message Broker**
2. **Cài đặt Celery**
3. **Tạo ứng dụng Celery đầu tiên**
4. **Chạy Celery Worker**
5. **Gọi tác vụ (Calling Tasks)**

---

### 1. Chọn một Message Broker

#### Message Broker là gì?
- Celery cần một **message broker** (hàng đợi tin nhắn) để gửi và nhận tác vụ giữa ứng dụng và worker.
- Broker phổ biến:
  - **RabbitMQ**: Ổn định, mạnh mẽ, được khuyến nghị.
  - **Redis**: Nhẹ, dễ cài đặt, phù hợp cho dự án nhỏ.

#### Lựa chọn
- Chúng ta sẽ dùng **Redis** vì dễ cài đặt và phù hợp cho người mới bắt đầu.
- Cài Redis:
  - Tải và chạy Redis trên máy (mặc định: `localhost:6379`):
  ```bash
  redis-server
  ```

---

### 2. Cài đặt Celery

#### Cài đặt qua pip
- Mở terminal và chạy:
```bash
pip install celery
```
- Nếu dùng Redis làm broker:
```bash
pip install celery redis
```

#### Kiểm tra cài đặt
- Chạy lệnh sau để kiểm tra phiên bản Celery:
```bash
celery --version
```
- Kết quả ví dụ: `5.3.6` (tùy phiên bản bạn cài).

---

### 3. Tạo ứng dụng Celery đầu tiên

#### Tạo file tác vụ
Tạo file `tasks.py` với nội dung sau:
```python
# tasks.py
from celery import Celery

# Khởi tạo ứng dụng Celery với Redis làm broker
app = Celery('tasks', broker='redis://localhost:6379/0')

# Định nghĩa một tác vụ đơn giản
@app.task
def add(x, y):
    return x + y
```
- **Giải thích**:
  - `Celery('tasks')`: Tạo ứng dụng Celery với tên `tasks`.
  - `broker='redis://localhost:6379/0'`: Kết nối đến Redis tại `localhost:6379`, dùng database 0.
  - `@app.task`: Định nghĩa hàm `add` là một tác vụ Celery.

---

### 4. Chạy Celery Worker

#### Worker là gì?
- Worker là tiến trình xử lý các tác vụ được gửi đến broker.

#### Chạy worker
- Mở terminal, vào thư mục chứa `tasks.py`, và chạy:
```bash
celery -A tasks worker --loglevel=info
```
- **Giải thích**:
  - `-A tasks`: Chỉ định file `tasks.py` chứa ứng dụng Celery.
  - `worker`: Khởi động worker.
  - `--loglevel=info`: Hiển thị thông tin chi tiết khi worker chạy.

#### Kết quả mong đợi
- Bạn sẽ thấy thông báo như:
```
[2025-04-01 12:00:00,000: INFO] celery.worker: Starting worker...
[2025-04-01 12:00:00,001: INFO] Tasks: add
```
- Worker đã sẵn sàng xử lý tác vụ.

---

### 5. Gọi tác vụ (Calling Tasks)

#### Gọi tác vụ bất đồng bộ
- Mở một terminal khác (giữ worker chạy), chạy Python shell:
```bash
python
```
- Trong shell, nhập:
```python
from tasks import add

# Gọi tác vụ bất đồng bộ
result = add.delay(4, 6)
```
- **Giải thích**:
  - `add.delay(4, 6)`: Gửi tác vụ `add` với tham số `4` và `6` đến Redis. Worker sẽ xử lý bất đồng bộ.
  - `result`: Một đối tượng `AsyncResult` để theo dõi trạng thái.

#### Kiểm tra kết quả
- Trong shell, kiểm tra trạng thái và kết quả:
```python
print(result.status)  # Trạng thái: PENDING, STARTED, SUCCESS
print(result.get())   # Lấy kết quả: 10
```
- Worker trong terminal kia sẽ ghi log:
```
[2025-04-01 12:01:00,000: INFO] Task tasks.add[...] succeeded in 0.1s: 10
```

#### Gọi tác vụ đồng bộ (nếu cần)
- Bạn cũng có thể chạy trực tiếp mà không qua worker:
```python
print(add(4, 6))  # Kết quả: 10
```
- Nhưng mục đích chính của Celery là xử lý bất đồng bộ, nên thường dùng `.delay()`.

---

### Ví dụ đầy đủ

#### File `tasks.py`
```python
from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task
def add(x, y):
    return x + y
```

#### Terminal 1: Chạy worker
```bash
celery -A tasks worker --loglevel=info
```

#### Terminal 2: Gọi tác vụ
```bash
python
```
```python
from tasks import add
result = add.delay(5, 7)
print(result.get())  # Kết quả: 12
```

---

### Một số mẹo bổ sung

#### Tùy chỉnh cấu hình
- Bạn có thể thêm backend để lưu kết quả:
```python
app = Celery('tasks',
             broker='redis://localhost:6379/0',
             backend='redis://localhost:6379/1')
```
- `backend`: Lưu kết quả tác vụ vào Redis database 1.

#### Kiểm tra trạng thái chi tiết
- Dùng `AsyncResult` để kiểm tra:
```python
from celery import result
task_id = result.id  # Lấy ID tác vụ
task = result.AsyncResult(task_id)
print(task.state)    # SUCCESS, FAILURE, v.v.
print(task.result)   # 12
```

#### Debug lỗi
- Nếu worker không chạy:
  - Kiểm tra Redis có chạy không (`redis-cli ping` trả về `PONG`).
  - Đảm bảo đường dẫn broker đúng.

---

### Tóm tắt
- **Broker**: Chọn Redis (`redis://localhost:6379/0`).
- **Cài đặt**: `pip install celery[redis]`.
- **Tác vụ**: Định nghĩa trong `tasks.py` với `@app.task`.
- **Worker**: Chạy bằng `celery -A tasks worker`.
- **Gọi**: Dùng `.delay()` để gửi tác vụ bất đồng bộ.

Dưới đây là hướng dẫn chi tiết bằng tiếng Việt về phần **Configuration** trong tài liệu **"First Steps with Celery"** tại https://docs.celeryq.dev/en/stable/getting-started/first-steps-with-celery.html#configuration. Tôi sẽ giải thích rõ ràng các khái niệm, cách cấu hình, và cung cấp ví dụ thực tế để bạn dễ hiểu.

---

### Tổng quan về Configuration trong Celery

Phần **Configuration** giải thích cách tùy chỉnh ứng dụng Celery để kiểm soát hành vi của nó, như cách gửi tác vụ, lưu kết quả, hoặc giới hạn thời gian thực thi. Celery cho phép cấu hình thông qua nhiều cách: trực tiếp trong mã nguồn, qua file cấu hình, hoặc biến môi trường.

#### Nội dung chính:
1. **Cách cấu hình Celery**
2. **Các thuộc tính cấu hình phổ biến**
3. **Ví dụ thực tế**

---

### 1. Cách cấu hình Celery

Celery cung cấp ba cách chính để cấu hình:

#### a. Cấu hình trực tiếp trong mã nguồn
- Thêm các tham số khi khởi tạo ứng dụng Celery:
```python
from celery import Celery

app = Celery('tasks',
             broker='redis://localhost:6379/0',
             backend='redis://localhost:6379/1',
             task_serializer='json',
             result_expires=3600)
```
- **Giải thích**:
  - `broker`: Địa chỉ Redis làm message broker.
  - `backend`: Nơi lưu kết quả (Redis database 1).
  - `task_serializer`: Định dạng dữ liệu gửi đi (JSON).
  - `result_expires`: Thời gian kết quả được lưu (3600 giây = 1 giờ).

#### b. Sử dụng đối tượng cấu hình
- Tạo một dictionary hoặc file riêng để quản lý cấu hình:
```python
# tasks.py
from celery import Celery

app = Celery('tasks')

# Cấu hình qua dictionary
app.conf.update(
    broker_url='redis://localhost:6379/0',
    result_backend='redis://localhost:6379/1',
    task_serializer='json',
    accept_content=['json'],
    result_serializer='json',
    timezone='Asia/Ho_Chi_Minh',
    enable_utc=True,
)
```
- **Giải thích**:
  - `app.conf.update`: Cập nhật cấu hình bằng dictionary.
  - `timezone`: Đặt múi giờ (ví dụ: Việt Nam).
  - `enable_utc`: Đồng bộ thời gian với UTC.

#### c. Sử dụng file cấu hình riêng
- Tạo file `celeryconfig.py`:
```python
# celeryconfig.py
broker_url = 'redis://localhost:6379/0'
result_backend = 'redis://localhost:6379/1'
task_serializer = 'json'
accept_content = ['json']
result_serializer = 'json'
timezone = 'Asia/Ho_Chi_Minh'
enable_utc = True
```
- Load file này vào ứng dụng:
```python
# tasks.py
from celery import Celery

app = Celery('tasks')
app.config_from_object('celeryconfig')
```
- **Lợi ích**: Tách cấu hình ra khỏi mã nguồn, dễ bảo trì.

---

### 2. Các thuộc tính cấu hình phổ biến

Dưới đây là một số thuộc tính quan trọng bạn có thể tùy chỉnh:

#### a. Broker và Backend
- `broker_url`: Địa chỉ broker (ví dụ: `'redis://localhost:6379/0'`).
- `result_backend`: Nơi lưu kết quả (ví dụ: `'redis://localhost:6379/1'`).

#### b. Serialization (Định dạng dữ liệu)
- `task_serializer`: Định dạng dữ liệu gửi đi (mặc định: `'json'`, hỗ trợ `'pickle'`, `'yaml'`).
- `accept_content`: Loại nội dung worker chấp nhận (`['json']` để chỉ dùng JSON).
- `result_serializer`: Định dạng kết quả trả về (`'json'`).

#### c. Thời gian và múi giờ
- `timezone`: Múi giờ cho lịch trình (ví dụ: `'Asia/Ho_Chi_Minh'`).
- `enable_utc`: Bật/tắt UTC (mặc định: `True`).
- `result_expires`: Thời gian lưu kết quả (giây, mặc định: 1 ngày).

#### d. Giới hạn tác vụ
- `task_time_limit`: Thời gian tối đa để hoàn thành (giây).
- `task_soft_time_limit`: Thời gian mềm (nếu vượt quá, báo lỗi nhưng không dừng ngay).
- Ví dụ:
```python
app.conf.task_time_limit = 300  # 5 phút
app.conf.task_soft_time_limit = 240  # 4 phút
```

#### e. Giới hạn worker
- `worker_concurrency`: Số lượng worker chạy đồng thời (mặc định: số CPU).
- Ví dụ:
```python
app.conf.worker_concurrency = 4  # 4 worker cùng lúc
```

---

### 3. Ví dụ thực tế

#### File đầy đủ với cấu hình
```python
# tasks.py
from celery import Celery

app = Celery('tasks')

# Cấu hình
app.conf.update(
    broker_url='redis://localhost:6379/0',
    result_backend='redis://localhost:6379/1',
    task_serializer='json',
    accept_content=['json'],
    result_serializer='json',
    timezone='Asia/Ho_Chi_Minh',
    enable_utc=True,
    task_time_limit=300,  # Tác vụ tối đa 5 phút
    task_soft_time_limit=240,  # Cảnh báo sau 4 phút
)

# Tác vụ ví dụ
@app.task
def add(x, y):
    import time
    time.sleep(10)  # Giả lập tác vụ lâu
    return x + y
```

#### Chạy worker
- Mở terminal:
```bash
celery -A tasks worker --loglevel=info
```

#### Gọi tác vụ
- Mở terminal khác, chạy Python shell:
```bash
python
```
```python
from tasks import add
result = add.delay(5, 7)
print(result.get(timeout=20))  # Chờ tối đa 20 giây, kết quả: 12
```

#### Kiểm tra cấu hình
- Nếu tác vụ chạy quá 5 phút (`task_time_limit`), worker sẽ dừng và báo lỗi:
```
[ERROR] Task tasks.add[...] raised exception: TimeLimitExceeded()
```

#### Thử với file cấu hình riêng
- Tạo `celeryconfig.py`:
```python
# celeryconfig.py
broker_url = 'redis://localhost:6379/0'
result_backend = 'redis://localhost:6379/1'
timezone = 'Asia/Ho_Chi_Minh'
task_time_limit = 300
```
- Sửa `tasks.py`:
```python
# tasks.py
from celery import Celery

app = Celery('tasks')
app.config_from_object('celeryconfig')

@app.task
def add(x, y):
    return x + y
```
- Chạy lại worker và gọi tác vụ như trên – kết quả tương tự.

---

### Một số lưu ý

- **Ưu tiên JSON**: Dùng `json` thay vì `pickle` cho `task_serializer` để bảo mật (tránh chạy mã độc).
- **Kiểm tra cấu hình**: In cấu hình hiện tại trong shell:
```python
from tasks import app
print(app.conf.humanize())
```
- **Debug lỗi**:
  - Nếu worker không nhận tác vụ: Kiểm tra `broker_url` có đúng không (Redis có chạy không?).
  - Nếu kết quả không lưu: Đảm bảo `result_backend` được cấu hình.

---

### Tóm tắt
- **Cấu hình trực tiếp**: Thêm vào `Celery()` hoặc dùng `app.conf.update`.
- **File riêng**: Dùng `app.config_from_object` với `celeryconfig.py`.
- **Thuộc tính phổ biến**: Broker, backend, serializer, timezone, time limits.
- **Ví dụ**: Giới hạn thời gian, đặt múi giờ Việt Nam.

Hy vọng phần giải thích này giúp bạn nắm rõ **Configuration** trong Celery! Nếu bạn muốn thử thêm cấu hình phức tạp (như lập lịch với Celery Beat), hãy cho tôi biết nhé!