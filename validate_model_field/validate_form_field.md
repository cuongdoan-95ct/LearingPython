# Validation trong Django Form và Model

**Django Form sẽ tự động sử dụng các validators được định nghĩa trong Model**, nhưng có một số điểm quan trọng cần hiểu rõ:

## 1. Cách Form sử dụng Model Validators

Khi bạn tạo một ModelForm, Django sẽ tự động:

```python
from django import forms
from .models import User

class UserForm(forms.ModelForm):
    class Meta:
        model = User
        fields = ['username', 'password']
        
    # Form sẽ tự động kế thừa:
    # - Tất cả validators từ model field
    # - Các ràng buộc như max_length, unique, blank, null...
```

## 2. Cơ chế hoạt động

1. **Khi gọi `form.is_valid()`**:
   - Form sẽ tự động chạy tất cả validators được định nghĩa trong model field
   - Các validator như `validate_password_strength` sẽ được thực thi

2. **Thứ tự validation**:
   ```python
   form.clean()          # Form-level validation
     → form.clean_<field>()  # Field-level validation
       → model.clean()       # Model-level validation
         → model field validators  # Validators định nghĩa trong model
   ```

## 3. Ví dụ cụ thể

Giả sử model của bạn:

```python
from django.core.validators import MinLengthValidator

class User(models.Model):
    username = models.CharField(
        max_length=30,
        validators=[MinLengthValidator(4)],
        unique=True
    )
```

Form sẽ tự động:
- Áp dụng validator `MinLengthValidator(4)`
- Kiểm tra ràng buộc `max_length=30`
- Thực hiện unique check (thông qua query)

## 4. Những điểm quan trọng

1. **Form sẽ KHÔNG tự động gọi `model.full_clean()`**:
   - Chỉ các field validators được áp dụng
   - Nếu muốn chạy model's `clean()` method, bạn cần thủ công:

     ```python
     class UserForm(forms.ModelForm):
         def clean(self):
             cleaned_data = super().clean()
             self.instance = User(**cleaned_data)
             self.instance.clean()  # Gọi model's clean() thủ công
             return cleaned_data
     ```

2. **Unique validation hoạt động khác biệt**:
   - ModelForm sẽ thực hiện SELECT query để check unique
   - Không phải đợi tới khi save mới kiểm tra

3. **Bạn có thể override validators**:
   ```python
   class UserForm(forms.ModelForm):
       username = forms.CharField(validators=[MyCustomValidator()])  # Ghi đè validators
   ```

## 5. Khi nào cần thêm validation ở Form?

Nên thêm validation ở Form khi:
- Cần rules đặc biệt chỉ áp dụng cho form đó
- Muốn có error messages khác với model
- Cần validate mối quan hệ giữa các fields

Ví dụ:
```python
class RegistrationForm(UserForm):
    password_confirm = forms.CharField()
    
    def clean(self):
        data = super().clean()
        if data['password'] != data['password_confirm']:
            raise forms.ValidationError("Passwords don't match")
        return data
```

## 6. Best Practices

1. **Định nghĩa core validators ở model**:
   - Đảm bảo tính nhất quán toàn hệ thống
   - Áp dụng cho mọi cách tạo/modify object

2. **Thêm form-specific validators ở Form**:
   - Cho các yêu cầu đặc thù từng form
   - Customize error messages

3. **Luôn sử dụng ModelForm cho CRUD operations**:
   - Tận dụng tự động validation từ model
   - Giảm duplicate code

## Tóm tắt

✅ **Django Form DÙNG validators từ Model**:
- Tự động áp dụng field validators
- Tự động kiểm tra các ràng buộc như max_length, unique

⚠️ **Nhưng KHÔNG tự động gọi**:
- Model's `clean()` method
- Validation phụ thuộc vào nhiều fields

Bạn nên kết hợp cả hai để có hệ thống validation mạnh mẽ và linh hoạt.