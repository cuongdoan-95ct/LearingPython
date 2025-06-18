```python
from rest_framework import serializers

class BaseDetailResponseSerializer(serializers.Serializer):
    data = serializers.DictField()
```

```python
class NotificationSerializer(serializers.Serializer):
    id = serializers.CharField()
    is_read = serializers.BooleanField()
    message = serializers.CharField()

class NotificationDetailSerializer(BaseDetailResponseSerializer):
    data = NotificationSerializer()
```

```python
class CategorySerializer(serializers.Serializer):
    id = serializers.CharField()
    title = serializers.CharField()
    description = serializers.CharField()

class CategoryDetailSerializer(BaseDetailResponseSerializer):
    data = CategorySerializer()
```


```python
from drf_spectacular.extensions import OpenApiSerializerExtension
from rest_framework import serializers

class CustomDataWrapperSerializer(serializers.Serializer):
    data = serializers.DictField()

class CustomDataWrapper(OpenApiSerializerExtension):
    target_class = 'path.to.serializers.BaseDetailResponseSerializer'

    def get_name(self):
        return "DataWrapper"

    def get_schema(self, direction):
        schema = super().get_schema(direction)
        return {
            "type": "object",
            "properties": {
                "data": schema
            }
        }
```


```python
SPECTACULAR_SETTINGS = {
    'EXTENSIONS': [
        'path.to.schemas.CustomDataWrapper',
    ],
}
```

```python
from rest_framework.viewsets import ModelViewSet
from .models import Notification, Category
from .serializers import NotificationDetailSerializer, CategoryDetailSerializer

class NotificationViewSet(ModelViewSet):
    queryset = Notification.objects.all()
    serializer_class = NotificationDetailSerializer

class CategoryViewSet(ModelViewSet):
    queryset = Category.objects.all()
    serializer_class = CategoryDetailSerializer
```