# Throttling trong Django REST Framework (DRF)

## 🧠 Throttling là gì?

**Throttling** là cơ chế giới hạn số lượng request mà một client (người dùng, IP, token...) có thể gửi trong một khoảng thời gian nhất định.  
Nó giúp bảo vệ API khỏi bị **lạm dụng**, **spam**, hoặc **DDoS nhẹ**.

---

## ✅ Ví dụ dễ hiểu:

> Bạn muốn mỗi người dùng **chỉ được gọi API 100 lần mỗi ngày**. DRF sẽ từ chối những request vượt quá giới hạn và trả về HTTP 429 (Too Many Requests).

---

## 1. Kích hoạt throttling trong `settings.py`

```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.UserRateThrottle',
        'rest_framework.throttling.AnonRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'user': '10/day',
        'anon': '100/hour',
    }
}
```

### 👉 Giải thích:
| Key       | Ý nghĩa                          |
|-----------|----------------------------------|
| `user`    | Đối với người dùng đã đăng nhập  |
| `anon`    | Đối với người chưa đăng nhập     |
| `10/day`  | Tối đa 10   request mỗi ngày     |

---

## 🛠️ 2. DRF cung cấp các lớp throttle mặc định:

| Lớp | Mô tả |
|-----|------|
| `AnonRateThrottle` | Dựa trên IP của request nếu user chưa login |
| `UserRateThrottle` | Dựa trên `request.user` nếu user đã login |

---

## 📦 3. Gắn throttle cho từng view hoặc toàn API

### Cách 1: Throttling toàn cục (qua `settings.py`) ✅

Bạn đã làm ở bước trên.

### Cách 2: Throttling riêng cho từng View 👇

```python
from rest_framework.views import APIView
from rest_framework.throttling import UserRateThrottle

class MyThrottle(UserRateThrottle):
    rate = '5/min'  # Tối đa 5 request mỗi phút

class MyView(APIView):
    throttle_classes = [MyThrottle]

    def get(self, request):
        return Response({"message": "Hello"})
```

---

## 🧪 4. DRF xử lý ra sao khi vượt giới hạn?

Nếu người dùng vượt quá số lần cho phép, DRF sẽ tự động trả về:

```json
HTTP 429 Too Many Requests

{
  "detail": "Request was throttled. Expected available in 12 seconds."
}
```

Bạn có thể custom lại thông điệp nếu muốn.

---

## 🧱 5. Viết Throttle class tùy chỉnh

```python
from rest_framework.throttling import BaseThrottle

class IPBlockThrottle(BaseThrottle):
    def allow_request(self, request, view):
        blocked_ips = ['192.168.1.10']
        return request.META['REMOTE_ADDR'] not in blocked_ips

    def wait(self):
        return None
```

---

## 🧠 Khi nào nên dùng throttling?

| Tình huống                             | Có nên dùng throttling? |
|----------------------------------------|--------------------------|
| Public API không cần auth              | ✅ Nên (chống spam IP)   |
| API người dùng đăng nhập               | ✅ Nên (giới hạn abuse)  |
| Nội bộ, không public, có auth tốt      | ❌ Không cần thiết       |

---

## 🧠 So sánh: **Authentication / Permission / Throttle**

| Cơ chế         | Dùng để...                          |
|----------------|-------------------------------------|
| Authentication | Ai đang gửi request?                |
| Permission     | Họ có quyền làm điều này không?     |
| Throttle       | Họ được gửi bao nhiêu request?      |

---

## 🎯 Tổng kết

- Dễ cấu hình (`DEFAULT_THROTTLE_CLASSES`)
- Có sẵn cho anonymous và user login
- Có thể tùy chỉnh riêng từng View
- Dùng tốt cho public API để bảo vệ server

---

Bạn muốn mình hướng dẫn thêm phần **viết middleware chống spam**, hay **phân biệt throttling theo IP, Token, User group**, không? Mình làm luôn cho chi tiết 😎.