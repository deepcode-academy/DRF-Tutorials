# 🌐 Setting Up a Django Project and DRF

# 📌 Set up the environment (Virtual Environment)

## 📌 Create a virtual environment:

```shell
python -m venv env
```

## 📌 Activate the virtual environment:
### Windows

```shell
env\Scripts\activate
```

### Linux/macOS:

```shell
source venv/bin/activate
```

## 📌 Install Django and DRF:

```shell
pip install django
```

```shell
pip install djangorestframework
```

# 📁 Create Django Project and App

## 📌 Create Django project:

```shell
django-admin startproject config .
cd config
```

## 📌 Create Django app:

```shell
python manage.py startapp api
```

# ⚙️ Configure settings.py

## 📌 Add apps to INSTALLED_APPS:

```python
INSTALLED_APPS = [
    # ...
    'rest_framework',
    'api',
]
```

# 📦 Create a Model (e.g. Post model)

### 📌 Inside api/models.py:

```python
from django.db import models

class Post(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()

    def __str__(self):
        return self.title
```

🧠 Explanation:
- `models.Model` – bu `Post` class Django model ekanini bildiradi.
- `title = models.CharField(...)` – bu sarlavha uchun maxsus ustun bo‘lib, 100ta belgigacha matn qabul qiladi.
- `content = models.TextField()` – bu post matni uchun, uzun matnlarni saqlashga mo‘ljallangan.
- `__str__` – admin panelda model obyektining ko‘rinishini o‘zgartiradi (sarlavhani ko‘rsatadi).

## 🔗 Make and apply migrations:

```shell
python manage.py makemigrations
python manage.py migrate
```

# 🧰 Create a Serializer

### 📌 Inside api/serializers.py:

```python
from rest_framework import serializers
from .models import Post

class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = '__all__'
```

# 🌐 Create a View

### 📌 Inside api/views.py:

```python
from rest_framework import viewsets
from .models import Post
from .serializers import PostSerializer

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```

# 🛣️ Setup URLs

### 📌 api/urls.py (create this file):

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import PostViewSet

router = DefaultRouter()
router.register(r'posts', PostViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

### 📌 Update config/urls.py

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('api.urls')),
]
```

# 🚀 Run the Server

```shell
python manage.py runserver
```

# 🔍 Open in browser:

```shell
http://127.0.0.1:8000/api/posts/
```
