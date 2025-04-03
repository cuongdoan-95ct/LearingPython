### 📌 **Canvas trong Celery – Điều phối các Task một cách linh hoạt**  

**Canvas** trong Celery là một tập hợp các công cụ giúp bạn **kết hợp, sắp xếp và điều phối các task** một cách linh hoạt. Nó giúp bạn xây dựng các workflow phức tạp, chẳng hạn như:  
- Chạy nhiều task song song.  
- Thực hiện các task theo thứ tự nhất định.  
- Xử lý task theo nhóm, chuỗi hoặc rẽ nhánh.  

---

## 🔹 **1. Các thành phần chính của Canvas trong Celery**  

Canvas có nhiều thành phần mạnh mẽ, nhưng quan trọng nhất gồm:  
1. **Signature** – Định nghĩa một task với tham số cố định.  
2. **Chain** – Chuỗi task chạy tuần tự, task sau dùng kết quả của task trước.  
3. **Group** – Chạy nhiều task song song.  
4. **Chord** – Kết hợp Group và Callback.  
5. **Map, Starmap** – Chạy một task trên danh sách giá trị.  
6. **Chunks** – Chia dữ liệu lớn thành các phần nhỏ hơn để xử lý song song.  

---

## 🔹 **2. Signature – Định nghĩa Task với Tham số Cố Định**  
**Signature** giúp tạo một task với tham số được đặt trước mà không cần chạy ngay.  

📌 **Ví dụ:**  
```python
from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task
def add(x, y):
    return x + y

# Tạo một task với tham số cố định
add_signature = add.s(2, 3)  # Giống như add(2, 3), nhưng chưa chạy
result = add_signature.apply_async()  # Thực thi task
print(result.get())  # Output: 5
```
💡 **Lợi ích:** Giúp tạo task linh hoạt mà không cần thực thi ngay.  

---

## 🔹 **3. Chain – Chuỗi Task Chạy Tuần Tự**  
**Chain** giúp bạn chạy các task theo thứ tự, task sau sẽ nhận kết quả của task trước làm đầu vào.  

📌 **Ví dụ:**  
```python
from celery import chain

chain_task = chain(add.s(2, 3), add.s(4))  # add(2, 3) → add(result, 4)
result = chain_task.apply_async()
print(result.get())  # Output: 9 (5 + 4)

#or
# Tạo chuỗi: (4 + 6) * 2
chain = add.s(4, 6) | add.s(2)
result = chain.apply_async()
print(result.get())  # 12

```
#### Ví dụ thực tế
```python
@app.task
def square(x):
    return x * x

# Chuỗi: (4 + 6) -> bình phương
chain = add.s(4, 6) | square.s()
result = chain.delay()
print(result.get())  # 100 (10 * 10)
```

💡 **Ứng dụng:** Phù hợp cho các quy trình có tính phụ thuộc, như pipeline xử lý dữ liệu.  

---

#### Ý nghĩa
- Chain hữu ích khi cần xử lý tuần tự, ví dụ: tải dữ liệu → xử lý → lưu kết quả.

---

## 🔹 **4. Group – Chạy Nhiều Task Song Song**  
**Group** cho phép bạn chạy nhiều task **đồng thời** và nhận kết quả của tất cả task đó.  

📌 **Ví dụ:**  
```python
from celery import group

group_task = group(add.s(2, 3), add.s(4, 5), add.s(6, 7))
result = group_task.apply_async()
print(result.get())  # Output: [5, 9, 13]
```
💡 **Ứng dụng:** Phù hợp khi cần chạy nhiều task độc lập song song (ví dụ: tải dữ liệu từ nhiều nguồn).  

---

## 🔹 **5. Chord – Kết hợp Group và Callback**  
**Chord** là một biến thể của **Group**, trong đó có một task được chạy **sau khi tất cả task trong nhóm hoàn thành**.  

📌 **Ví dụ:**  
```python
from celery import chord

@app.task
def sum_of_list(numbers):
    return sum(numbers)

chord_task = chord((add.s(2, 3), add.s(4, 5)),  # Các task chạy song song
    sum_of_list.s()  # Task cuối cùng chạy sau khi các task trên hoàn thành
)
result = chord_task.apply_async()
print(result.get())  # Output: 14 (5 + 9)
```
💡 **Ứng dụng:** Rất hữu ích cho các bài toán tổng hợp kết quả từ nhiều task song song.  

---

## 🔹 **6. Map và Starmap – Lặp qua danh sách giá trị**  
Nếu bạn muốn chạy một task với một danh sách giá trị khác nhau, bạn có thể dùng **map** hoặc **starmap**.  

📌 **Ví dụ với starmap:**  
```python
add_task = add.starmap([(2, 3), (4, 5), (6, 7)])
result = add_task.apply_async()
print(result.get())  # Output: [5, 9, 13]
```
💡 **Ứng dụng:** Xử lý dữ liệu hàng loạt một cách hiệu quả.  

---

## 🔹 **7. Chunks – Chia dữ liệu lớn thành phần nhỏ**  
**Chunks** giúp chia nhỏ danh sách dữ liệu thành từng phần nhỏ và xử lý song song.  

📌 **Ví dụ:**  
```python
from celery import chunks

chunk_task = chunks(add.s(), [(1, 2), (3, 4), (5, 6)], 2)  # Mỗi nhóm 2 phần tử
result = chunk_task.apply_async()
print(result.get())  # Output: [[3, 7], [11]]
```
💡 **Ứng dụng:** Hữu ích khi làm việc với tập dữ liệu lớn.  

---

## 🔹 **8.Tóm tắt nhanh các thành phần của Canvas**
| Công cụ | Chức năng | Ví dụ |
|---------|----------|-------|
| **Signature** | Tạo một task với tham số cố định | `add.s(2, 3)` |
| **Chain** | Chạy task theo chuỗi | `chain(add.s(2, 3), add.s(4))` |
| **Group** | Chạy nhiều task song song | `group(add.s(2, 3), add.s(4, 5))` |
| **Chord** | Kết hợp Group và Callback | `chord([...], callback_task)` |
| **Map** | Áp dụng task cho danh sách giá trị | `add.map([(2, 3), (4, 5)])` |
| **Starmap** | Giống map nhưng truyền tham số khác nhau | `add.starmap([(2, 3), (4, 5)])` |
| **Chunks** | Chia dữ liệu lớn thành phần nhỏ | `chunks(add.s(), [(1,2), (3,4)], 2)` |

---

### 🚀 **Kết luận**
Canvas trong Celery giúp bạn **xây dựng workflow mạnh mẽ** với các công cụ như **Chain, Group, Chord** để xử lý công việc **tuần tự, song song hoặc kết hợp cả hai**. Đây là một trong những tính năng quan trọng giúp Celery trở thành một hệ thống xử lý background task mạnh mẽ.  



Tôi hiểu rằng bạn muốn tôi tiếp tục giải thích nội dung của phần **"The Primitives"** trong tài liệu Celery tại https://docs.celeryq.dev/en/stable/userguide/canvas.html#the-primitives bằng tiếng Việt. Dưới đây là hướng dẫn chi tiết, bao quát toàn bộ nội dung của phần này, với giải thích và ví dụ cụ thể để bạn dễ hiểu và áp dụng.

---

### Tổng quan về "The Primitives" trong Canvas

**Canvas** trong Celery là công cụ mạnh mẽ để tổ chức các tác vụ (tasks) thành các luồng công việc phức tạp. Phần **"The Primitives"** giới thiệu các khối xây dựng cơ bản (primitives) để kết hợp tasks, như chuỗi, nhóm, hoặc ánh xạ.

#### Nội dung chính:
1. **Signature (Ký hiệu - `task.s()`)**
2. **Chain (Chuỗi - `|`)**
3. **Group (Nhóm - `group`)**
4. **Chord (Hợp âm - `chord`)**
5. **Map (Ánh xạ - `map`)**
6. **Starmap (Ánh xạ sao - `starmap`)**
7. **Chunks (Chia khối - `chunks`)**

---

### 1. Signature (Ký hiệu - `task.s()`)

#### Signature là gì?
- Signature là cách tạo một "bản sao" của task để sử dụng trong Canvas mà không thực thi ngay lập tức. Nó giống như một kế hoạch (blueprint) cho task.

#### Ví dụ cơ bản
```python
# tasks.py
from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task
def add(x, y):
    return x + y

# Tạo signature
sig = add.s(4, 6)
result = sig.delay()  # Gọi bất đồng bộ
print(result.get())   # Kết quả: 10
```

#### Dùng với tùy chọn
```python
sig = add.s(4, 6)
result = sig.apply_async(countdown=5)  # Chạy sau 5 giây
print(result.get())  # 10
```

#### Ý nghĩa
- Signature là nền tảng cho các primitives khác, cho phép trì hoãn và kết hợp tasks.

---

### 3. Group (Nhóm - `group`)

#### Group là gì?
- Group chạy nhiều task song song và trả về danh sách kết quả.

#### Ví dụ cơ bản
```python
from celery import group

# Nhóm: [1+1, 2+2, 3+3]
g = group(add.s(i, i) for i in range(1, 4))
result = g.delay()
print(result.get())  # [2, 4, 6]
```

#### Ví dụ thực tế
```python
@app.task
def process_item(item):
    return f"Xử lý {item}"

items = ["A", "B", "C"]
g = group(process_item.s(item) for item in items)
result = g.apply_async()
print(result.get())  # ['Xử lý A', 'Xử lý B', 'Xử lý C']
```

#### Ý nghĩa
- Group phù hợp khi cần xử lý đồng thời nhiều tác vụ độc lập, như gửi email cho nhiều người.

---

### 4. Chord (Hợp âm - `chord`)

#### Chord là gì?
- Chord là sự kết hợp của Group và một callback task. Group chạy song song, sau đó callback xử lý tất cả kết quả.

#### Ví dụ cơ bản
```python
from celery import chord

# Nhóm: [1+1, 2+2, 3+3] -> Tổng kết quả
c = chord(group(add.s(i, i) for i in range(1, 4)))(add.s(0))
result = c.delay()
print(result.get())  # 12 (2 + 4 + 6)
```

#### Ví dụ thực tế
```python
@app.task
def sum_results(results):
    return sum(results)

# Chord: Tính tổng các bình phương
c = chord([square.s(i) for i in [2, 3, 4]])(sum_results.s())
result = c.apply_async()
print(result.get())  # 29 (4 + 9 + 16)
```

#### Ý nghĩa
- Chord hữu ích khi cần xử lý song song rồi tổng hợp kết quả, như tính toán dữ liệu lớn.

---

### 5. Map (Ánh xạ - `map`)

#### Map là gì?
- Map áp dụng một task lên từng phần tử trong danh sách, tương tự hàm `map()` trong Python.

#### Ví dụ cơ bản
```python
# Ánh xạ: Nhân đôi từng số
result = add.map([1, 2, 3]).delay(1)  # add(x, 1) cho từng phần tử
print(result.get())  # [2, 3, 4]
```

#### Ví dụ thực tế
```python
@app.task
def process_string(s):
    return s.upper()

result = process_string.map(['a', 'b', 'c']).apply_async()
print(result.get())  # ['A', 'B', 'C']
```

#### Ý nghĩa
- Map đơn giản hóa việc áp dụng task lên danh sách, nhưng chỉ dùng một tham số.

---

### 6. Starmap (Ánh xạ sao - `starmap`)

#### Starmap là gì?
- Starmap giống Map, nhưng áp dụng task lên danh sách các bộ tham số (tuples).

#### Ví dụ cơ bản
```python
# Starmap: Áp dụng add cho từng cặp
result = add.starmap([(1, 2), (3, 4), (5, 6)]).delay()
print(result.get())  # [3, 7, 11]
```

#### Ví dụ thực tế
```python
@app.task
def concat(a, b):
    return f"{a}-{b}"

pairs = [("x", "1"), ("y", "2")]
result = concat.starmap(pairs).apply_async()
print(result.get())  # ['x-1', 'y-2']
```

#### Ý nghĩa
- Starmap hữu ích khi task cần nhiều tham số từ danh sách.

---

### 7. Chunks (Chia khối - `chunks`)

#### Chunks là gì?
- Chunks chia danh sách lớn thành các khối nhỏ (chunks) và áp dụng task lên từng khối song song.

#### Ví dụ cơ bản
```python
# Chia [1, 2, 3, 4] thành 2 khối, mỗi khối 2 phần tử
result = add.chunks([(1, 1), (2, 2), (3, 3), (4, 4)], 2).delay()
print(result.get())  # [[2, 4], [6, 8]]
```

#### Ví dụ thực tế
```python
@app.task
def process_batch(items):
    return [item * 2 for item in items]

data = [1, 2, 3, 4, 5, 6]
result = process_batch.chunks([(x,) for x in data], 3).apply_async()
print(result.get())  # [[2, 4, 6], [8, 10, 12]]
```

#### Ý nghĩa
- Chunks tối ưu khi xử lý dữ liệu lớn, phân chia công việc cho nhiều worker.

---

### Tóm tắt và So sánh

| Primitive     | Mô tả                          | Ví dụ kết quả         |
|---------------|--------------------------------|-----------------------|
| **Signature** | Tạo bản sao task               | `add.s(4, 6)` -> 10  |
| **Chain**     | Chạy tuần tự                  | `(4+6)*2` -> 20      |
| **Group**     | Chạy song song                | `[1+1, 2+2]` -> [2, 4] |
| **Chord**     | Nhóm + Callback               | `[1+1, 2+2]` -> 6    |
| **Map**       | Áp dụng lên danh sách         | `[1, 2]` -> [2, 3]   |
| **Starmap**   | Áp dụng lên danh sách cặp     | `[(1, 2), (3, 4)]` -> [3, 7] |
| **Chunks**    | Chia khối và xử lý song song  | `[(1, 1), (2, 2)]` -> [[2], [4]] |

---

### Ví dụ tổng hợp

```python
# tasks.py
from celery import Celery, group, chord, chain

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task
def add(x, y):
    return x + y

@app.task
def multiply(x, y):
    return x * y

@app.task
def sum_all(results):
    return sum(results)

# Chain
c = chain(add.s(4, 6) | multiply.s(2))()
print(c.get())  # 20

# Group
g = group(add.s(i, i) for i in range(3))()
print(g.get())  # [0, 2, 4]

# Chord
ch = chord(group(add.s(i, i) for i in range(3)))(sum_all.s())()
print(ch.get())  # 6

# Starmap
s = add.starmap([(1, 2), (3, 4)])()
print(s.get())  # [3, 7]
```

#### Chạy worker
```bash
celery -A tasks worker --loglevel=info
```

---

### Ý nghĩa thực tế
- **Chain**: Xử lý tuần tự (tải file → phân tích → lưu).
- **Group**: Gửi email hàng loạt.
- **Chord**: Tính toán song song rồi tổng hợp (ví dụ: phân tích dữ liệu).
- **Chunks**: Xử lý dữ liệu lớn (như xử lý 1 triệu bản ghi).


---

### Tổng quan về Stamping trong Celery

**Stamping** là một tính năng nâng cao trong Celery, cho phép bạn "đóng dấu" (stamp) các tác vụ hoặc nhóm tác vụ với dữ liệu bổ sung (metadata). Dữ liệu này có thể được sử dụng để theo dõi, xác thực, hoặc điều khiển hành vi của task trong quá trình thực thi.

#### Nội dung chính:
1. **Stamping là gì?**
2. **Cách sử dụng Stamping**
3. **Ví dụ thực tế**
4. **Ứng dụng của Stamping**

---
Dưới đây là hướng dẫn chi tiết bằng tiếng Việt về phần **"Stamping"** trong tài liệu Celery tại https://docs.celeryq.dev/en/stable/userguide/canvas.html#stamping. Phần này giải thích cách sử dụng tính năng **stamping** (đóng dấu) trong Celery để gắn thông tin bổ sung vào các tác vụ (tasks) hoặc luồng công việc (workflows) trong Canvas. Tôi sẽ giải thích từng phần kèm ví dụ cụ thể để bạn hiểu rõ.

---

### Tổng quan về Stamping trong Celery

**Stamping** là một tính năng nâng cao trong Celery, cho phép bạn "đóng dấu" (stamp) các tác vụ hoặc nhóm tác vụ với dữ liệu bổ sung (metadata). Dữ liệu này có thể được sử dụng để theo dõi, xác thực, hoặc điều khiển hành vi của task trong quá trình thực thi.

#### Nội dung chính:
1. **Stamping là gì?**
2. **Cách sử dụng Stamping**
3. **Ví dụ thực tế**
4. **Ứng dụng của Stamping**

---

### 1. Stamping là gì?

#### Khái niệm
- **Stamping** gắn một tập hợp dữ liệu (gọi là "stamp") vào task hoặc workflow trước khi chúng được gửi đến worker.
- Stamp này được truyền qua toàn bộ luồng công việc và có thể truy cập trong task thông qua `self.request`.
- Dùng để:
  - Theo dõi nguồn gốc task.
  - Gắn thông tin phiên (session) hoặc người dùng.
  - Điều khiển logic task dựa trên stamp.

#### Ví dụ cơ bản về ý tưởng
- Bạn muốn gắn ID người dùng vào một task để biết ai đã khởi tạo nó.

---

### 2. Cách sử dụng Stamping

#### Cách đóng dấu
- Sử dụng phương thức `.stamp()` trên signature hoặc workflow.
- Stamp là một dictionary chứa dữ liệu bạn muốn gắn.

#### Cú pháp cơ bản
```python
# tasks.py
from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task(bind=True)
def process_data(self, data):
    stamp = self.request.stamps  # Truy cập stamp
    print(f"Stamp: {stamp}")
    return f"Xử lý {data}"

# Tạo signature và stamp
sig = process_data.s("Hello")
sig.stamp(stamps={'user_id': 123, 'session': 'abc'})
result = sig.delay()
print(result.get())  # Xử lý Hello
```

#### Kết quả trong worker
- Worker sẽ in:
```
Stamp: {'user_id': 123, 'session': 'abc'}
```

---

### 3. Ví dụ thực tế

#### a. Stamping trên Signature đơn
```python
@app.task(bind=True)
def log_task(self, message):
    stamps = self.request.stamps
    user_id = stamps.get('user_id', 'unknown')
    print(f"User {user_id}: {message}")
    return message.upper()

# Đóng dấu
sig = log_task.s("Xin chào")
sig.stamp(stamps={'user_id': 456})
result = sig.apply_async()
print(result.get())  # XIN CHÀO
```
- Worker in: `User 456: Xin chào`.

#### b. Stamping trên Workflow (Chain)
```python
@app.task
def add(x, y):
    return x + y

@app.task(bind=True)
def multiply(self, x):
    stamps = self.request.stamps
    factor = stamps.get('factor', 1)
    return x * factor

# Tạo chuỗi và stamp
workflow = (add.s(4, 6) | multiply.s())
workflow.stamp(stamps={'factor': 3})
result = workflow.delay()
print(result.get())  # 30 (10 * 3)
```
- Stamp `factor` được truyền từ `add` sang `multiply`.

#### c. Stamping trên Group
```python
from celery import group

@app.task(bind=True)
def process_item(self, item):
    stamps = self.request.stamps
    batch_id = stamps.get('batch_id', 'no_batch')
    return f"{batch_id}: {item}"

# Nhóm và stamp
g = group(process_item.s(i) for i in ['A', 'B', 'C'])
g.stamp(stamps={'batch_id': 'batch_001'})
result = g.apply_async()
print(result.get())  # ['batch_001: A', 'batch_001: B', 'batch_001: C']
```

#### d. Stamping với Chord
```python
from celery import chord

@app.task
def sum_results(results):
    return sum(results)

@app.task(bind=True)
def square(self, x):
    stamps = self.request.stamps
    print(f"Stamp trong square: {stamps}")
    return x * x

# Chord và stamp
c = chord([square.s(i) for i in [1, 2, 3]])(sum_results.s())
c.stamp(stamps={'workflow_id': 'calc_001'})
result = c.delay()
print(result.get())  # 14 (1 + 4 + 9)
```
- Worker in: `Stamp trong square: {'workflow_id': 'calc_001'}` cho từng task.

---

### 4. Ứng dụng của Stamping

#### a. Theo dõi nguồn gốc
- Gắn ID người dùng hoặc phiên để biết ai khởi tạo task:
```python
sig = process_data.s("Test")
sig.stamp(stamps={'user_id': 789, 'timestamp': '2025-04-03'})
result = sig.delay()
```

#### b. Điều khiển logic
- Dùng stamp để thay đổi hành vi task:
```python
@app.task(bind=True)
def conditional_task(self, data):
    stamps = self.request.stamps
    mode = stamps.get('mode', 'normal')
    if mode == 'debug':
        print(f"Debug: {data}")
    return data

sig = conditional_task.s("Kiểm tra")
sig.stamp(stamps={'mode': 'debug'})
sig.delay()  # In: Debug: Kiểm tra
```

#### c. Quản lý luồng công việc lớn
- Gắn thông tin dự án hoặc batch vào toàn bộ workflow:
```python
g = group(add.s(i, i) for i in range(3))
g.stamp(stamps={'project': 'MathProj', 'batch': '2025Q2'})
g.apply_async()
```

---

### Một số lưu ý

#### Truy cập Stamps
- Dùng `self.request.stamps` trong task có `bind=True`.
- Nếu không có stamp, `stamps` là dictionary rỗng `{}`.

#### Truyền Stamps
- Stamp được truyền qua toàn bộ workflow (chain, group, chord) trừ khi bị ghi đè.

#### Ghi đè Stamps
- Có thể thêm hoặc ghi đè stamp ở bước sau:
```python
sig = add.s(4, 6)
sig.stamp(stamps={'step': 1})
sig.stamp(stamps={'step': 2, 'extra': 'new'})  # Ghi đè 'step'
```

---

### Tóm tắt

- **Stamping**: Gắn metadata vào task/workflow bằng `.stamp(stamps={...})`.
- **Truy cập**: Dùng `self.request.stamps` trong task.
- **Ứng dụng**: Theo dõi, điều khiển logic, quản lý workflow.
- **Hỗ trợ**: Signature, Chain, Group, Chord.

#### Ví dụ tổng hợp
```python
# tasks.py
from celery import Celery, group, chain

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task(bind=True)
def step1(self, x):
    stamps = self.request.stamps
    print(f"Step1 stamps: {stamps}")
    return x + 1

@app.task(bind=True)
def step2(self, x):
    stamps = self.request.stamps
    factor = stamps.get('factor', 1)
    print(f"Step2 stamps: {stamps}")
    return x * factor

# Workflow với stamp
workflow = chain(step1.s(5) | step2.s())
workflow.stamp(stamps={'user': 'Nam', 'factor': 3})
result = workflow.delay()
print(result.get())  # 18 ((5 + 1) * 3)
```
- Worker in:
```
Step1 stamps: {'user': 'Nam', 'factor': 3}
Step2 stamps: {'user': 'Nam', 'factor': 3}
```

#### Chạy worker
```bash
celery -A tasks worker --loglevel=info
```