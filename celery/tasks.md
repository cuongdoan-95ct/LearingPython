### Tổng quan về Tasks trong Celery
#### Nội dung chính:
1. **Cơ bản về Tasks (Basics)**
2. **Đặt tên Tasks (Task Names)**
3. **Tự động đặt tên (Autonaming)**
4. **Gọi Tasks (Calling Tasks)**
5. **Tác vụ tuần hoàn (Periodic Tasks)**
6. **Trạng thái Tasks (Task States)**
7. **Tùy chỉnh lớp Task (Custom Task Classes)**
8. **Các thuộc tính Task (Task Attributes)**
9. **Cách Tasks được gửi (How Tasks Are Sent)**
10. **Tác vụ từ xa (Remote Tasks)**

---

### 1. Cơ bản về Tasks (Basics)****

#### Task là gì?
- Task là đơn vị công việc mà Celery xử lý bất đồng bộ. Bạn định nghĩa task như một hàm Python và Celery gửi nó đến worker qua message broker.

#### Ví dụ cơ bản
```python
# tasks.py
from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task
def add(x, y):
    return x + y
```
- **Chạy worker**:
```bash
celery -A tasks worker --loglevel=info
```
- **Gọi task**:
```python
from tasks import add
result = add.delay(4, 6)
print(result.get())  # 10
```

#### Ghi log trong Task
- Dùng `self.request` để truy cập thông tin task:
```python
@app.task(bind=True)
def log_task(self, x):
    self.request.logger.info(f"Đang xử lý {x}")
    return x * 2
```

---

### 2. Đặt tên Tasks (Task Names)

#### Tùy chỉnh tên Task
- Mặc định, tên task là `<module>.<function>` (ví dụ: `tasks.add`).
- Tùy chỉnh bằng `name`:
```python
@app.task(name='tinh_tong')
def add(x, y):
    return x + y

app.send_task('tinh_tong', args=(4, 6))  # Kết quả: 10
```

#### Lợi ích
- Dễ quản lý khi tích hợp với hệ thống khác hoặc gọi từ xa.

---

### 3. Tự động đặt tên (Autonaming)

#### Cơ chế tự động
- Nếu không chỉ định `name`, Celery tự tạo tên dựa trên module và hàm.
- Ví dụ: Trong `myapp/tasks.py`, task `add` sẽ có tên `myapp.tasks.add`.

#### Kiểm tra tên
```python
print(add.name)  # tasks.add
```

#### Vô hiệu hóa autonaming
- Dùng `app.conf.task_autoname_tasks = False` (ít dùng).

---

### 4. Gọi Tasks (Calling Tasks)

#### Các cách gọi
- **`.delay()`**: Gọi bất đồng bộ với tham số vị trí:
```python
add.delay(4, 6)
```
- **`.apply_async()`**: Linh hoạt hơn:
```python
add.apply_async(args=(4, 6), countdown=10)  # Chạy sau 10 giây
add.apply_async(kwargs={'x': 4, 'y': 6})   # Dùng tham số tên
```
- **Gọi trực tiếp**: Không qua worker:
```python
add(4, 6)  # 10
```
- **Gửi bằng tên (send_task)**: Không cần import task:
```python
app.send_task('tasks.add', args=(4, 6))
```

#### Liên kết Tasks (Linking)
- Gọi task khác khi hoàn thành:
```python
@app.task
def multiply(x, y):
    return x * y

add.apply_async((4, 6), link=multiply.s(2))  # (4 + 6) * 2 = 20
```

#### Nhóm Tasks (Groups)
- Chạy nhiều task song song:
```python
from celery import group

g = group(add.s(i, i) for i in range(5))  # [0+0, 1+1, 2+2, 3+3, 4+4]
result = g.delay()
print(result.get())  # [0, 2, 4, 6, 8]
```

#### Chuỗi Tasks (Chains)
- Chạy tuần tự:
```python
from celery import chain

workflow = chain(add.s(4, 6) | multiply.s(3))  # (4 + 6) * 3
result = workflow.delay()
print(result.get())  # 30
```

#### Chord (Nhóm + Callback)
- Chạy nhóm rồi gọi callback:
```python
from celery import chord

c = chord([add.s(i, i) for i in range(3)])(multiply.s(2))  # ([0+0, 1+1, 2+2]) * 2
print(c.get())  # [0, 2, 4] * 2 = 8 (tổng rồi nhân)
```

---

### 5. Tác vụ tuần hoàn (Periodic Tasks)

#### Định nghĩa
- Task chạy định kỳ theo lịch (dùng Celery Beat).
- Cấu hình trong `beat_schedule`:
```python
from celery import Celery
from celery.schedules import crontab

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task
def periodic_task():
    print("Chạy mỗi 30 giây")

app.conf.beat_schedule = {
    'run-every-30-seconds': {
        'task': 'tasks.periodic_task',
        'schedule': 30.0,  # 30 giây
        # 'schedule': crontab(minute='*/5'),  # Mỗi 5 phút
    },
}
```
- **Chạy Beat**:
```bash
celery -A tasks beat --loglevel=info
```
- Worker phải chạy cùng lúc để xử lý.

---

### 6. Trạng thái Tasks (Task States)

#### Trạng thái mặc định
- `PENDING`, `STARTED`, `SUCCESS`, `FAILURE`, `RETRY`.

#### Tùy chỉnh trạng thái
```python
@app.task(bind=True)
def progress_task(self):
    self.update_state(state='PROGRESS', meta={'percent': 50})
    import time
    time.sleep(5)
    return "Xong"

result = progress_task.delay()
print(result.status)  # PROGRESS
print(result.info)    # {'percent': 50}
```

#### Kiểm tra trạng thái
```python
print(result.state)   # SUCCESS
print(result.result)  # Xong
```

#### Trạng thái tùy chỉnh
- Định nghĩa thêm trạng thái:
```python
from celery.states import state, SUCCESS

@app.task(bind=True)
def custom_state_task(self):
    self.update_state(state=state('WAITING'), meta={'info': 'Đang chờ'})
    return "OK"
```

---

### 7. Tùy chỉnh lớp Task (Custom Task Classes)

#### Kế thừa `Task`
```python
from celery import Celery, Task

app = Celery('tasks', broker='redis://localhost:6379/0')

class MyTask(Task):
    def on_success(self, retval, task_id, args, kwargs):
        print(f"Thành công: {retval}")
    def on_failure(self, exc, task_id, args, kwargs):
        print(f"Thất bại: {exc}")
    def before_start(self, task_id, args, kwargs):
        print(f"Bắt đầu: {task_id}")

@app.task(base=MyTask)
def add(x, y):
    return x + y
```

#### Ví dụ thực tế
```python
@app.task(base=MyTask)
def risky_task(x):
    if x < 0:
        raise ValueError("Số âm")
    return x * 2

risky_task.delay(5)   # In: Bắt đầu -> Thành công: 10
risky_task.delay(-1)  # In: Bắt đầu -> Thất bại: Số âm
```

---

### 8. Các thuộc tính Task (Task Attributes)

#### Thuộc tính phổ biến
- **`bind`**: Truyền `self` vào task.
- **`ignore_result`**: Không lưu kết quả:
```python
@app.task(ignore_result=True)
def log_only(x):
    print(x)
```
- **`max_retries`**: Số lần thử lại:
```python
@app.task(max_retries=3)
def retry_task():
    raise Exception("Lỗi")
```
- **`default_retry_delay`**: Thời gian chờ giữa các lần thử:
```python
@app.task(default_retry_delay=60)  # 1 phút
def retry_later():
    raise Exception("Thử lại sau")
```
- **`rate_limit`**: Giới hạn tần suất:
```python
@app.task(rate_limit='10/m')  # 10 task/phút
def limited_task():
    return "OK"
```

#### Ví dụ tổng hợp
```python
@app.task(bind=True, max_retries=3, default_retry_delay=5, rate_limit='5/m')
def complex_task(self):
    try:
        raise ValueError("Lỗi thử nghiệm")
    except Exception as e:
        self.retry(exc=e)
```

---

### 9. Cách Tasks được gửi (How Tasks Are Sent)

#### Cơ chế
- Task được gửi qua broker dưới dạng message (JSON, Pickle, v.v.).
- Cấu hình serialization:
```python
app.conf.task_serializer = 'json'
app.conf.accept_content = ['json']
```

#### Sự kiện (Events)
- Worker gửi sự kiện để giám sát:
```bash
celery -A tasks worker --loglevel=info --events
```
- Dùng `celery events` để xem:
```bash
celery -A tasks events
```

---

### 10. Tác vụ từ xa (Remote Tasks)

#### Gửi task đến queue cụ thể
```python
@app.task(queue='high_priority')
def urgent_task():
    return "Khẩn cấp"

urgent_task.apply_async(args=(), queue='high_priority')
```
- Cấu hình worker cho queue:
```bash
celery -A tasks worker -Q high_priority --loglevel=info
```

#### Gửi đến nhiều worker
- Dùng `routing_key`:
```python
add.apply_async((4, 6), routing_key='low_priority')
```

---

### Tóm tắt đầy đủ

- **Cơ bản**: Task là hàm với `@app.task`.
- **Đặt tên**: Tùy chỉnh hoặc tự động (`name` hoặc module-based).
- **Gọi**: `.delay()`, `.apply_async()`, nhóm, chuỗi, chord.
- **Tuần hoàn**: Dùng `beat_schedule` với Celery Beat.
- **Trạng thái**: Theo dõi và tùy chỉnh (`update_state`).
- **Tùy chỉnh**: Kế thừa `Task` để thêm logic.
- **Thuộc tính**: `bind`, `max_retries`, `rate_limit`, v.v.
- **Gửi**: Qua broker, có thể giám sát bằng events.
- **Từ xa**: Queue và routing.

#### Ví dụ tổng hợp
```python
from celery import Celery, Task, group
from celery.schedules import crontab

app = Celery('tasks', broker='redis://localhost:6379/0')

class LogTask(Task):
    def on_success(self, retval, task_id, args, kwargs):
        print(f"Xong {task_id}: {retval}")

@app.task(base=LogTask, bind=True, max_retries=2)
def process_data(self, data):
    try:
        if not data:
            raise ValueError("Rỗng")
        return f"Xử lý: {data}"
    except Exception as e:
        self.retry(countdown=5)

app.conf.beat_schedule = {
    'run-periodic': {
        'task': 'tasks.process_data',
        'schedule': crontab(minute='*/1'),  # Mỗi phút
        'args': ('Hello',),
    },
}

# Gọi
process_data.delay("Test")
g = group(process_data.s(i) for i in ["A", "B", "C"])
g.delay()
```

### 📌 **Task Request (`self.request`) trong Celery**  

Trang tài liệu bạn gửi nói về **Task Request Context**, tức là thông tin về một task khi nó đang chạy trong Celery. Đây là cách Celery cung cấp **metadata** về task, bao gồm **ID của task, số lần retry, hàng đợi (queue), thông tin routing**, v.v.  

---

## 🔹 **1. Cách truy cập `self.request` trong một task**  
Để truy cập **ngữ cảnh yêu cầu (`self.request`)**, bạn cần dùng **`bind=True`** khi khai báo task:  
```python
from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task(bind=True)
def my_task(self, x, y):
    print(f"📌 Task ID: {self.request.id}")  
    print(f"🔄 Số lần retry: {self.request.retries}")  
    print(f"📩 Task chạy trên queue: {self.request.delivery_info['routing_key']}")  
    return x + y
```
📌 **Giải thích:**  
- `self.request.id`: ID duy nhất của task.  
- `self.request.retries`: Số lần retry của task.  
- `self.request.delivery_info['routing_key']`: Xác định queue mà task đang chạy.  

---

## 🔹 **2. Các thuộc tính quan trọng của `self.request`**  
| 🏷️ Thuộc tính | 🔍 Mô tả |
|--------------|--------|
| `self.request.id` | ID duy nhất của task. |
| `self.request.retries` | Số lần task đã được retry. |
| `self.request.is_eager` | `True` nếu task đang chạy trực tiếp mà không qua worker. |
| `self.request.hostname` | Tên worker đang thực thi task. |
| `self.request.delivery_info` | Thông tin routing của task, bao gồm queue name. |
| `self.request.args` | Danh sách tham số truyền vào task (dạng `tuple`). |
| `self.request.kwargs` | Các tham số dạng `keyword arguments` (dạng `dict`). |
| `self.request.headers` | Các headers đi kèm khi gửi task. |

---

## 🔹 **3. Ví dụ: Kiểm soát số lần retry của task bằng `self.request.retries`**  
Nếu bạn muốn một task **tự động retry khi gặp lỗi**, có thể dùng `self.request.retries` để kiểm tra số lần retry và `self.retry()` để thử lại:  
```python
@app.task(bind=True, max_retries=3)
def risky_task(self):
    try:
        # Giả lập lỗi
        raise ValueError("❌ Lỗi xảy ra!")
    except Exception as exc:
        print(f"🔄 Task retry lần {self.request.retries + 1}...")  
        raise self.retry(exc=exc, countdown=5)  
```
📌 **Giải thích:**  
- Nếu task gặp lỗi, nó sẽ **tự động retry tối đa 3 lần**.  
- `self.request.retries` giúp kiểm tra số lần retry hiện tại.  
- `countdown=5` đảm bảo mỗi lần retry cách nhau **5 giây**.  

---

## 🔹 **4. Ứng dụng `self.request` trong thực tế**
✅ **Ghi log thông tin task** để debug.  
✅ **Theo dõi số lần retry** để xử lý lỗi tốt hơn.  
✅ **Xác định queue hoặc worker đang thực thi** để điều phối task.  

Bạn có muốn thử áp dụng `self.request` vào một trường hợp cụ thể không? 🚀


### 📌 **Semipredicates trong Celery**

Trong Celery, **semipredicates** là một khái niệm liên quan đến cách định nghĩa các điều kiện trong **retry** của task (thử lại khi có lỗi). Chúng được sử dụng để **kiểm tra các lỗi cụ thể** có nên retry task hay không.

---

## 🔹 **1. Khái niệm về Semipredicates**

**Semipredicates** là một cách đơn giản để **xác định điều kiện** khi một task gặp lỗi và quyết định xem có nên retry (thử lại) hay không. Celery cho phép bạn sử dụng **các predicate (biểu thức logic)** để kiểm tra loại lỗi mà task gặp phải và chỉ retry trong trường hợp lỗi đó là một lỗi cụ thể.

---

## 🔹 **2. Cách sử dụng Semipredicates**

Semipredicates giúp bạn chỉ retry các task khi gặp lỗi mà bạn **có thể kiểm soát được**, thay vì retry tất cả các lỗi. Cách này giúp tránh retry vô hạn đối với các lỗi không phải là vấn đề tạm thời, như lỗi không tìm thấy file hoặc lỗi hệ thống nghiêm trọng.

### 📌 **Ví dụ: Chỉ retry khi gặp lỗi `ConnectionError`**
```python
from celery import Celery
from celery.exceptions import SoftTimeLimitExceeded
from time import sleep

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task(bind=True, max_retries=3, autoretry_for=(ConnectionError,))
def fetch_data(self):
    try:
        # Giả lập lỗi kết nối
        raise ConnectionError("🔌 Mất kết nối mạng!")
    except ConnectionError as exc:
        print(f"🔄 Retry vì lỗi: {exc}")
        raise self.retry(exc=exc, countdown=5)  # Retry sau 5 giây
```

### 🔹 **Giải thích**:
- **`autoretry_for=(ConnectionError,)`**: Chỉ retry khi gặp lỗi `ConnectionError`.
- **`self.retry()`**: Thử lại sau 5 giây nếu gặp lỗi `ConnectionError`.
- **`max_retries=3`**: Retry tối đa 3 lần.

---

## 🔹 **3. Định nghĩa Semipredicate Tùy chỉnh**

Celery cũng hỗ trợ việc tạo ra **semipredicate tùy chỉnh**. Thay vì chỉ retry khi gặp một loại lỗi nhất định, bạn có thể định nghĩa một predicate của riêng bạn để kiểm tra lỗi.

### 📌 **Ví dụ: Semipredicate tùy chỉnh**
```python
def custom_predicate(exc):
    # Retry chỉ khi lỗi là "Mất kết nối"
    return isinstance(exc, ConnectionError) or isinstance(exc, TimeoutError)

@app.task(bind=True, max_retries=3, autoretry_for=(Exception,), retry_kwargs={'max_retries': 3})
def fetch_with_custom_predicate(self):
    try:
        raise ConnectionError("Lỗi kết nối!")
    except Exception as exc:
        if custom_predicate(exc):
            print(f"Retrying due to error: {exc}")
            raise self.retry(exc=exc, countdown=5)
        else:
            print(f"Not retrying due to error: {exc}")
            raise exc  # Không retry nếu không phải lỗi cần retry
```

### 🔹 **Giải thích**:
- **`custom_predicate(exc)`**: Hàm tùy chỉnh kiểm tra loại lỗi. Nó chỉ trả về `True` khi gặp `ConnectionError` hoặc `TimeoutError`.
- **`autoretry_for=(Exception,)`**: Tất cả các loại lỗi đều sẽ thử lại, nhưng predicate tùy chỉnh quyết định có retry hay không.
- **`retry(exc=exc, countdown=5)`**: Retry sau 5 giây nếu predicate trả về `True`.

---

## 🔹 **4. Lợi ích của Semipredicates**
- **Kiểm soát linh hoạt**: Bạn có thể chỉ retry những lỗi tạm thời (ví dụ: mất kết nối, lỗi timeout) thay vì retry tất cả các lỗi.
- **Giảm thiểu việc retry không cần thiết**: Tránh retry các lỗi nghiêm trọng mà không có khả năng phục hồi.
- **Tăng hiệu suất hệ thống**: Khi không retry các lỗi không cần thiết, bạn giảm tải cho hệ thống.

---

## 🔹 **5. Kết luận**
**Semipredicates** giúp bạn **tinh chỉnh quá trình retry** trong Celery, chỉ retry khi gặp các lỗi tạm thời hoặc những lỗi có thể phục hồi được, giúp tránh retry các lỗi không cần thiết và tối ưu hiệu suất của hệ thống.

Nếu bạn muốn retry task một cách thông minh, **semipredicates** là công cụ hữu ích cho bạn để **kiểm soát** lỗi và retry một cách chính xác.

Bạn có muốn thử áp dụng semipredicates trong dự án của mình để retry task một cách linh hoạt hơn không? 🚀