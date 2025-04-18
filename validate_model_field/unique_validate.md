# Validation cho Unique Field (username với unique=True)

Khi bạn đặt `unique=True` cho trường username, validation sẽ được thực hiện ở **cả serializer lẫn model**, nhưng ở các thời điểm và cách thức khác nhau:

## 1. Ở Serializer Level

DRF thực hiện unique check **trước khi tạo model instance**:

```python
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['username']
        extra_kwargs = {
            'username': {'validators': []}  # Mặc định DRF tự động thêm UniqueValidator
        }
```

- **Cách hoạt động**:
  - Khi gọi `serializer.is_valid()`, DRF sẽ thực hiện query kiểm tra uniqueness
  - Nếu username đã tồn tại → raise `ValidationError` ngay lập tức
  - Lỗi có format phù hợp cho API (HTTP 400)

- **Ưu điểm**:
  - Fail fast (phát hiện lỗi sớm)
  - Tránh phải tạo model instance
  - Error message thân thiện với API

## 2. Ở Model Level

Django thực hiện unique check **khi save model**:

```python
class User(models.Model):
    username = models.CharField(unique=True)
```

- **Cách hoạt động**:
  - Khi gọi `user.save()`, Django sẽ kiểm tra uniqueness
  - Nếu username đã tồn tại → raise `IntegrityError`
  - Lỗi này xuất phát từ database level

- **Ưu điểm**:
  - Bảo vệ ở tầng cuối cùng
  - Hoạt động với mọi cách tạo model (shell, admin, raw queries)

## 3. So sánh trực tiếp

| Aspect           | Serializer Validation | Model Validation |
|------------------|-----------------------|------------------|
| **Thời điểm**    | Trước khi tạo instance | Khi save model |
| **Error type**   | `rest_framework.exceptions.ValidationError` | `django.db.IntegrityError` |
| **Nguồn gốc**    | DRF thực hiện query check | Database constraint |
| **Performance**  | Thêm 1 query SELECT | Database tự kiểm tra |
| **Use case**     | Ưu tiên cho API | Bảo vệ toàn hệ thống |

## 4. Tại sao cần cả hai?

1. **Serializer check**:
   - Phát hiện sớm lỗi unique
   - Trả về error format đẹp cho API
   - Tránh unnecessary operations

2. **Model check**:
   - Bắt các trường hợp bypass serializer
   - Đảm bảo ràng buộc ở DB level
   - Bảo vệ khi có race condition

## 5. Race Condition Lưu ý

Ngay cả với cả 2 tầng validation, vẫn có thể xảy ra **race condition** khi:
- Hai requests cùng kiểm tra username không tồn tại
- Sau đó cùng thử tạo user với username đó

Giải pháp:
```python
class UserSerializer(serializers.ModelSerializer):
    def create(self, validated_data):
        try:
            return User.objects.create(**validated_data)
        except IntegrityError:
            raise serializers.ValidationError({'username': 'This username already exists'})
```

## 6. Tùy chỉnh behavior

Nếu muốn tắt unique check ở serializer (chỉ dùng model check):
```python
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['username']
        extra_kwargs = {
            'username': {
                'validators': []  # Tắt UniqueValidator mặc định
            }
        }
```

Nhưng cách này không recommended vì:
- Mất đi fail fast capability
- Error message từ DB ít thân thiện hơn
- Tăng khả năng gặp race condition

## Kết luận

Với `unique=True`, validation xảy ra ở **cả serializer lẫn model**:
- **Serializer**: Check trước để có UX tốt cho API
- **Model**: Last defense đảm bảo data integrity

Bạn nên giữ cả hai cơ chế này để có hệ thống mạnh mẽ và thân thiện với người dùng.