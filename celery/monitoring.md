Dưới đây là hướng dẫn chi tiết bằng tiếng Việt về nội dung của phần **"Monitoring and Management Guide"** trong tài liệu Celery tại https://docs.celeryq.dev/en/stable/userguide/monitoring.html. Phần này giải thích cách giám sát và quản lý các worker, task và hệ thống Celery. Tôi sẽ giải thích từng mục kèm ví dụ cụ thể để bạn dễ hiểu và áp dụng.

---

### Tổng quan về Monitoring trong Celery

Phần **Monitoring and Management Guide** cung cấp các công cụ và kỹ thuật để theo dõi trạng thái của worker, task, cũng như quản lý hiệu suất và xử lý sự cố trong hệ thống Celery.

#### Nội dung chính:
1. **Sự kiện (Events)**
2. **Công cụ giám sát (Monitoring Tools)**
   - celery events
   - celerymon
   - Flower
3. **Lệnh quản lý (Management Commands)**
4. **Giám sát trong mã nguồn (Programmatic Monitoring)**

---

### 1. Sự kiện (Events)

#### Sự kiện là gì?
- Celery tạo ra các **event** (sự kiện) mỗi khi có hoạt động quan trọng, như task bắt đầu, hoàn thành, hoặc worker khởi động.
- Các sự kiện này có thể được sử dụng để giám sát realtime.

#### Loại sự kiện
- **Task events**: `task-started`, `task-succeeded`, `task-failed`, `task-retried`.
- **Worker events**: `worker-online`, `worker-offline`, `worker-heartbeat`.

#### Bật sự kiện
- Khi chạy worker, thêm tùy chọn `-E` hoặc `--events`:
```bash
celery -A tasks worker --loglevel=info --events
```

#### Ví dụ cơ bản
```python
# tasks.py
from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task
def add(x, y):
    return x + y

add.delay(4, 6)
```
- Chạy worker với events:
```bash
celery -A tasks worker -E --loglevel=info
```
- Kết quả: Worker sẽ gửi sự kiện như:
```
[task: tasks.add started]
[task: tasks.add succeeded in 0.01s: 10]
```

---

### 2. Công cụ giám sát (Monitoring Tools)

#### a. `celery events`
- Công cụ dòng lệnh để xem sự kiện realtime.

##### Cách chạy
```bash
celery -A tasks events
```
- Hiển thị sự kiện như:
```
worker-online: worker1.example.com
task-received: tasks.add (uuid: 1234-...)
task-succeeded: tasks.add (result: 10)
```

##### Dump sự kiện
- In tất cả sự kiện dưới dạng JSON:
```bash
celery -A tasks events --dump
```
- Kết quả:
```json
{"type": "task-succeeded", "uuid": "1234-...", "result": 10}
```

#### b. `celerymon` (Đã lỗi thời)
- Trước đây là công cụ web để giám sát, nhưng hiện đã được thay bằng **Flower**.

#### c. Flower
- **Flower** là giao diện web mạnh mẽ để giám sát và quản lý Celery.

##### Cài đặt
```bash
pip install flower
```

##### Chạy Flower
```bash
celery -A tasks flower --port=5555
```
- Truy cập: `http://localhost:5555`
- Giao diện hiển thị:
  - Danh sách worker.
  - Trạng thái task (pending, running, completed).
  - Biểu đồ hiệu suất.

##### Ví dụ thực tế
- Chạy worker:
```bash
celery -A tasks worker --loglevel=info
```
- Chạy Flower:
```bash
celery -A tasks flower
```
- Gửi task:
```python
add.delay(5, 7)
```
- Trên Flower, bạn sẽ thấy task `tasks.add` với trạng thái và kết quả.

---

### 3. Lệnh quản lý (Management Commands)

#### Dùng `celery control` và `celery inspect`
- Điều khiển và kiểm tra worker từ xa.

##### a. `inspect`
- Xem thông tin worker:
```bash
celery -A tasks inspect active
```
- Kết quả:
```
worker1.example.com:
  - tasks.add[uuid:1234-...]
```

- Xem danh sách task đã đăng ký:
```bash
celery -A tasks inspect registered
```
- Kết quả:
```
worker1.example.com:
  - tasks.add
```

##### b. `control`
- Điều chỉnh worker:
- Giới hạn tần suất task:
```bash
celery -A tasks control rate_limit tasks.add 10/m
```
- Tắt worker từ xa:
```bash
celery -A tasks control shutdown
```

##### Ví dụ thực tế
- Chạy worker:
```bash
celery -A tasks worker --loglevel=info
```
- Kiểm tra worker đang hoạt động:
```bash
celery -A tasks inspect active
```
- Giới hạn `add` còn 5 task/phút:
```bash
celery -A tasks control rate_limit tasks.add 5/m
```

---

### 4. Giám sát trong mã nguồn (Programmatic Monitoring)

#### Sử dụng `celery.events`
- Lắng nghe sự kiện trong Python.

##### Ví dụ cơ bản
```python
from celery import Celery
from celery.events import EventReceiver

app = Celery('tasks', broker='redis://localhost:6379/0')

def my_monitor(app):
    def on_event(event):
        print(f"Sự kiện: {event['type']}")
        if event['type'] == 'task-succeeded':
            print(f"Task {event['uuid']} hoàn thành: {event['result']}")

    with app.connection() as conn:
        recv = EventReceiver(conn, handlers={'*': on_event})
        recv.capture(limit=None, timeout=None, wakeup=True)

if __name__ == '__main__':
    my_monitor(app)
```
- Chạy worker:
```bash
celery -A tasks worker -E
```
- Chạy monitor:
```bash
python tasks.py
```
- Gửi task:
```python
add.delay(3, 4)
```
- Kết quả monitor:
```
Sự kiện: task-received
Sự kiện: task-started
Sự kiện: task-succeeded
Task <uuid> hoàn thành: 7
```

#### Theo dõi trạng thái task
- Dùng `AsyncResult`:
```python
from celery.result import AsyncResult

result = add.delay(5, 5)
task = AsyncResult(result.id, app=app)
print(task.state)  # SUCCESS
print(task.result) # 10
```

#### Ví dụ thực tế
```python
@app.task(bind=True)
def long_task(self):
    self.update_state(state='PROGRESS', meta={'percent': 50})
    import time
    time.sleep(2)
    return "Xong"

result = long_task.delay()
task = AsyncResult(result.id, app=app)
print(task.state)   # PROGRESS
print(task.info)    # {'percent': 50}
print(task.get())   # Xong
```

---

### Tóm tắt

- **Sự kiện**: Theo dõi task/worker qua events (`-E`).
- **Công cụ**:
  - `celery events`: Xem realtime.
  - **Flower**: Giao diện web (http://localhost:5555).
- **Lệnh quản lý**:
  - `inspect`: Kiểm tra trạng thái.
  - `control`: Điều khiển worker.
- **Mã nguồn**: Dùng `EventReceiver` hoặc `AsyncResult` để giám sát.

#### Ví dụ tổng hợp
```python
# tasks.py
from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task
def add(x, y):
    return x + y

# Chạy worker với events
# celery -A tasks worker -E --loglevel=info

# Chạy Flower
# celery -A tasks flower

# Monitor trong mã nguồn
from celery.events import EventReceiver

def monitor_events(app):
    def on_event(event):
        print(f"Event: {event}")
    with app.connection() as conn:
        recv = EventReceiver(conn, handlers={'*': on_event})
        recv.capture(limit=None, timeout=None)

if __name__ == '__main__':
    monitor_events(app)
```

#### Thử nghiệm
1. Chạy worker:
```bash
celery -A tasks worker -E
```
2. Chạy Flower:
```bash
celery -A tasks flower
```
3. Gửi task:
```python
add.delay(8, 9)
```
4. Xem Flower hoặc chạy monitor để thấy sự kiện.

---

### Ứng dụng thực tế
- **Giám sát hiệu suất**: Dùng Flower để xem thời gian chạy task.
- **Debug**: Theo dõi lỗi qua `task-failed` events.
- **Quản lý**: Điều chỉnh worker khi tải cao.

Hy vọng hướng dẫn này giúp bạn hiểu rõ phần **Monitoring** trong Celery! Nếu cần thêm ví dụ hoặc giải thích sâu hơn, hãy cho tôi biết nhé!