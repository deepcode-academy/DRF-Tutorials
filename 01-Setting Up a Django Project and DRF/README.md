# 🧩 1-DARS: DJANGO LOYIHASI VA DRF SOZLASH

Bu darsda Django va Django REST Framework (DRF) yordamida loyiha yaratish va oddiy API sozlashni bosqichma-bosqich o'rganamiz. Har bir qadam tushunarli va aniq bo'ladi.

## ✅ 1. MUHITNI TAYYORLASH

### ❇️ **Python o'rnatish**:
📌 Python 3.8 yoki undan yuqori versiyani o'rnating (https://www.python.org/downloads/).

### ❇️ **Virtual muhit yaratish**:
📌 Virtual muhit loyiha uchun alohida muhit yaratadi, bu boshqa loyihalarga ta'sir qilmaslikni ta'minlaydi.
```bash
python -m venv env
source env/bin/activate  # Windows uchun: env\Scripts\activate
```

### ❇️ **Django va DRF o'rnatish**:
📌 Bu Django va DRF kutubxonalarini o'rnatadi.
```bash
pip install django djangorestframework
```

## ✅ 2. DJANGO LOYIHASINI YARATISH

### ❇️ **Loyiha yaratish**:
📌 Quyidagi buyruq `myproject` nomli yangi loyiha jildini yaratadi.
```bash
django-admin startproject myproject .
```

### ❇️ **Loyha tuzilishi**:
```
myproject/
├── manage.py
├── myproject/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
```
- `manage.py`: Loyiha boshqaruvi uchun asosiy fayl.
- `settings.py`: Loyiha sozlamalari.
- `urls.py`: URL marshrutlari.

### ❇️ **Serverni ishga tushirish**:
📌 Serverni ishga tushirib, loyiha ishlayotganini tekshiramiz.
```bash
python manage.py runserver
```
📌 Brauzerda `http://127.0.0.1:8000/` manziliga o'ting. "Django welcome" sahifasini ko'rasiz.

## ✅ 3. DJANGO ILOVASINI YARATISH
📌 Django loyihasida ilovalar (apps) alohida modullar sifatida ishlaydi.

### ❇️ **Ilova yaratish**:
```bash
python manage.py startapp myapp
```
📌 Bu `myapp` nomli ilova jildini yaratadi.

### ❇️ **Ilovani loyihaga qo'shish**:
📌 `myproject/settings.py` faylini oching va `INSTALLED_APPS` ro'yxatiga quyidagini qo'shing:
```python
INSTALLED_APPS = [
    ...
    'myapp',
    'rest_framework',  # DRF ni qo'shamiz
]
```

## ✅ 4. MA'LUMOTLAR BAZASINI SOZLASH
📌 Django standart ravishda SQLite bilan ishlaydi, bu oddiy loyihalar uchun yetarli.

### ❇️ **Model yaratish**:
📌 `myapp/models.py` faylida oddiy model yaratamiz:
```python
from django.db import models

class Item(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.name
```

### ❇️ **Migratsiyalarni yaratish va qo'llash**:
📌 Modelni ma'lumotlar bazasida jadvallar sifatida yaratish uchun:
```bash
python manage.py makemigrations
python manage.py migrate
```

## ✅ 5. DRF BILAN API YARATISH
📌 DRF yordamida oddiy API yaratamiz.

### ❇️ **Serializer yaratish**:
📌 `myapp/serializers.py` faylini yarating va quyidagi kodni qo'shing:
```python
from rest_framework import serializers
from .models import Item

class ItemSerializer(serializers.ModelSerializer):
    class Meta:
        model = Item
        fields = ['id', 'name', 'description', 'created_at']
```

### ❇️ **View yaratish**:
📌 `myapp/views.py` faylida API view yaratamiz:
```python
from rest_framework import viewsets
from .models import Item
from .serializers import ItemSerializer

class ItemViewSet(viewsets.ModelViewSet):
    queryset = Item.objects.all()
    serializer_class = ItemSerializer
```

### ❇️ **URL sozlash**:
📌 `myproject/urls.py` faylini yangilang:
```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from myapp.views import ItemViewSet

router = DefaultRouter()
router.register(r'items', ItemViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

## ✅ 6. API NI SINAB KO'RISH
📌 Serverni ishga tushiring:
```bash
python manage.py runserver
```
📌 Brauzerda `http://127.0.0.1:8000/items/` manziliga o'ting. DRF interfeysi orqali ma'lumotlarni ko'rishingiz, qo'shishingiz va o'zgartirishingiz mumkin.

## ✅ 7. MA'LUMOT QO'SHISH (IXTIYORIY)
📌 Django admin paneli orqali ma'lumot qo'shish uchun:

### ❇️ **Admin foydalanuvchisini yaratish**:
```bash
python manage.py createsuperuser
```

### ❇️ **Modelni admin panelida ro'yxatdan o'tkazish**:
📌 `myapp/admin.py` faylida modelni ro'yxatdan o'tkazing:
```python
from django.contrib import admin
from .models import Item

admin.site.register(Item)
```

📌 `http://127.0.0.1:8000/admin/` manzilida admin paneliga kiring va ma'lumot qo'shing.

