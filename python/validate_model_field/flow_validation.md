Khi gọi model.full_clear() -> clear_fields()  -> clear()  -> validate_unique()

Khi gọi form.is_valid() -> form.clean_<field>() -> form.clean() -> model.full_clean() (nếu gọi thủ công, mặc định sẽ không gọi) -> form.save() -> model.save()