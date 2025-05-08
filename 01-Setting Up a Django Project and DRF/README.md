# Setting Up a Django Project and DRF

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

### Inside api/models.py:

```python
from django.db import models

class Post(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()

    def __str__(self):
        return self.title
```

## 🔨 4.1. Make and apply migrations:

```shell
python manage.py makemigrations
python manage.py migrate
```

# 🧰 5. Create a Serializer

### Inside api/serializers.py:

```python
from rest_framework import serializers
from .models import Post

class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = '__all__'
```

