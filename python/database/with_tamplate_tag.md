Rất hay! Mình sẽ giúp bạn hiểu rõ về **`with` template tag** trong Django (tài liệu: [with – Django template tags](https://docs.djangoproject.com/en/5.1/ref/templates/builtins/#with)) — đây là một **cách gán biến tạm thời** trong template, giúp bạn viết code gọn hơn, dễ đọc hơn. Cùng tìm hiểu nha 👇

---

## 🧠 `with` là gì?

`{% with %}` là một **template tag** cho phép bạn **gán biến tạm thời trong template**, dùng để:

- Tránh lặp lại biểu thức dài
- Làm code gọn gàng, dễ hiểu hơn

---

## ✅ Cú pháp cơ bản:

```django
{% with new_var=original_var.subvalue %}
  {{ new_var }}
{% endwith %}
```

---

## 📌 Ví dụ đơn giản:

```django
{% with fullname=user.first_name|add:" "|add:user.last_name %}
  <p>Xin chào {{ fullname }}!</p>
{% endwith %}
```

**Tương đương với:**

```django
<p>Xin chào {{ user.first_name }} {{ user.last_name }}!</p>
```

Nhưng rõ ràng dễ đọc hơn nhiều.

---

## 📌 Gán nhiều biến một lúc:

```django
{% with name=user.name age=user.age %}
  <p>{{ name }} - {{ age }}</p>
{% endwith %}
```

---

## 🎯 Ứng dụng thực tế:

```django
{% for product in products %}
  {% with price=product.get_discount_price %}
    <p>{{ product.name }} - {{ price }} đ</p>
  {% endwith %}
{% endfor %}
```

👉 Gán `product.get_discount_price` vào biến `price` để không gọi lại nhiều lần.

---

## 🔒 Biến chỉ tồn tại trong `{% with %}`

Khi ra ngoài block `{% endwith %}`, biến đó **mất hiệu lực**:

```django
{% with x=5 %}
  {{ x }}  {# OK #}
{% endwith %}
{{ x }}  {# Sai: x không tồn tại #}
```

---

## ❓Khi nào nên dùng `with`?

| Nên dùng khi | Tránh dùng khi |
|-------------|----------------|
| Biểu thức dài, lồng nhiều thuộc tính | Chỉ cần gọi 1 lần |
| Dùng trong vòng lặp | Đặt biến toàn cục (vì biến chỉ sống trong block) |
| Đọc giá trị từ hàm (method) | Không cần tối ưu đọc |

---

## 🧪 Bonus – So sánh 2 cách

### ❌ Không dùng `with`:
```django
<p>{{ order.user.profile.avatar.url }}</p>
<p>{{ order.user.profile.avatar.url|default:'default.jpg' }}</p>
```

### ✅ Dùng `with`:
```django
{% with avatar=order.user.profile.avatar.url %}
  <p>{{ avatar }}</p>
  <p>{{ avatar|default:'default.jpg' }}</p>
{% endwith %}
```

➡ Dễ đọc hơn, ít lặp lại.

---

## ✅ Tóm gọn:

| Câu hỏi | Trả lời |
|---------|--------|
| `with` là gì? | Là template tag để gán biến tạm thời |
| Ưu điểm? | Gọn gàng, dễ đọc, tránh lặp |
| Dùng thế nào? | `{% with a=b %} ... {% endwith %}` |
| Biến có tồn tại ngoài block không? | ❌ Không |

---

## 🚀 Tóm tắt nhanh:

`{% with %}` **hoàn toàn có thể dùng lồng bên trong hoặc bên ngoài** `{% cache %}` block trong Django Template.

- Giúp bạn **tạo biến tạm** để truyền vào block cache
- Hoặc dùng để **tối ưu nội dung trong block cache**

---

## ✅ Cách dùng `with` + `cache`

### 🔸 Cách 1: Dùng `with` **trước khi cache**

```django
{% with cache_key="product_list_"|add:category.id|stringformat:"s" %}
    {% cache 300 cache_key %}
        {# Nội dung bạn muốn cache 5 phút #}
        <h2>{{ category.name }}</h2>
        {% for product in category.products.all %}
            <p>{{ product.name }}</p>
        {% endfor %}
    {% endcache %}
{% endwith %}
```

### ✅ Ưu điểm:
- Cho phép bạn build `cache_key` linh hoạt hơn
- Giúp bạn cache từng phần theo điều kiện cụ thể (ví dụ: mỗi `category.id`)

---

### 🔸 Cách 2: Dùng `with` **bên trong block cache** để code gọn hơn

```django
{% cache 300 "product_list" category.id %}
    {% with products=category.products.all %}
        {% for product in products %}
            <p>{{ product.name }}</p>
        {% endfor %}
    {% endwith %}
{% endcache %}
```

✅ Dùng trong trường hợp bạn không muốn lặp lại biểu thức `category.products.all` nhiều lần.

---

## 🔥 Ví dụ đầy đủ:

```django
{% with key="user_profile_"|add:user.id|stringformat:"s" %}
  {% cache 600 key %}
    {% with fullname=user.first_name|add:" "|add:user.last_name %}
      <h1>{{ fullname }}</h1>
      <p>Email: {{ user.email }}</p>
    {% endwith %}
  {% endcache %}
{% endwith %}
```

📌 Giải thích:

- Tạo `key` cache cho từng user profile
- Gán `fullname` tạm thời để tránh viết dài
- Cache nội dung đó trong 10 phút

---

## 📌 Lưu ý:

| Điều cần nhớ | Chi tiết |
|--------------|----------|
| `with` chỉ sống trong block của nó | Biến tạo bằng `{% with %}` không tồn tại ngoài `{% endwith %}` |
| `cache` có thể dùng nhiều tham số | `cache 300 "prefix" var1 var2` sẽ tạo key tự động |
| `with` rất hữu dụng khi tạo key thủ công | Đặc biệt khi kết hợp `.add`, filter, stringformat |

---

## 🎯 Kết luận

- `with` giúp bạn xây dựng `cache key` động hoặc tránh lặp code bên trong block cache.
- Kết hợp `with` + `cache` giúp viết template rõ ràng, tối ưu hiệu năng và bộ nhớ.

---

Bạn đang muốn cache phần nào cụ thể trong template không? Mình có thể gợi ý cấu trúc `with` + `cache` chuẩn nhất cho use case của bạn 💬