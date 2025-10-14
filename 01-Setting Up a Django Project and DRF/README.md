# 🌐 DJANGO REST FRAMEWORK ASOSLARI

# 🧩 1-DARS SETTING UP A DJANGO PROJECT AND DRF


## ✅ MUHITNI TAYYORLASH

### ❇️ **Virtual muhit yaratish**:

📌 Virtual muhit loyiha uchun alohida yaratadi, bu boshqa loyihalarga ta'sir qilmaslikni ta'minlaydi.

```shell
python -m venv env
env\Scripts\activate
```

### ❇️ **Django va DRF o'rnatish**:
```bash
pip install django djangorestframework
```
- Bu Django va DRF kutubxonalarini o'rnatadi.

## ✅ DJANGO LOYIHASINI YARATISH
📌 Django loyihasini boshlash uchun quyidagi qadamlar:

### ❇️ **Loyiha yaratish**:

```bash
django-admin startproject myproject .

```
📌 Bu `myproject` nomli yangi loyiha jildini yaratadi.

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
  
```bash
python manage.py runserver
```

📌 Brauzerda `http://127.0.0.1:8000/` manziliga o'ting. "Django welcome" sahifasini ko'rasiz.

## 3. Django ilovasini yaratish
Django loyihasida ilovalar (apps) alohida modullar sifatida ishlaydi.

- **Ilova yaratish**:
  ```bash
  python manage.py startapp myapp
  ```
  Bu `myapp` nomli ilova jildini yaratadi.

- **Ilovani loyihaga qo'shish**:
  `myproject/settings.py` faylini oching va `INSTALLED_APPS` ro'yxatiga quyidagini qo'shing:
  ```python
  INSTALLED_APPS = [
      ...
      'myapp',
      'rest_framework',  # DRF ni qo'shamiz
  ]
  ```

## 4. Ma'lumotlar bazasini sozlash
Django standart ravishda SQLite bilan ishlaydi, bu oddiy loyihalar uchun yetarli.

- **Model yaratish**:
  `myapp/models.py` faylida oddiy model yaratamiz:
  ```python
  from django.db import models

  class Item(models.Model):
      name = models.CharField(max_length=100)
      description = models.TextField()
      created_at = models.DateTimeField(auto_now_add=True)

      def __str__(self):
          return self.name
  ```

- **Migratsiyalarni yaratish va qo'llash**:
  ```bash
  python manage.py makemigrations
  python manage.py migrate
  ```
  Bu modelni ma'lumotlar bazasida jadvallar sifatida yaratadi.

## 5. DRF bilan API yaratish
DRF yordamida oddiy API yaratamiz.

- **Serializer yaratish**:
  `myapp/serializers.py` faylini yarating va quyidagi kodni qo'shing:
  ```python
  from rest_framework import serializers
  from .models import Item

  class ItemSerializer(serializers.ModelSerializer):
      class Meta:
          model = Item
          fields = ['id', 'name', 'description', 'created_at']
  ```

- **View yaratish**:
  `myapp/views.py` faylida API view yaratamiz:
  ```python
  from rest_framework import viewsets
  from .models import Item
  from .serializers import ItemSerializer

  class ItemViewSet(viewsets.ModelViewSet):
      queryset = Item.objects.all()
      serializer_class = ItemSerializer
  ```

- **URL sozlash**:
  `myproject/urls.py` faylini yangilang:
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

## 6. API ni sinab ko'rish
- Serverni ishga tushiring:
  ```bash
  python manage.py runserver
  ```
- Brauzerda `http://127.0.0.1:8000/items/` manziliga o'ting. DRF interfeysi orqali ma'lumotlarni ko'rishingiz, qo'shishingiz va o'zgartirishingiz mumkin.

## 7. Ma'lumot qo'shish (ixtiyoriy)
Django admin paneli orqali ma'lumot qo'shish uchun:

- **Admin foydalanuvchisini yaratish**:
  ```bash
  python manage.py createsuperuser
  ```
- `myapp/admin.py` faylida modelni ro'yxatdan o'tkazing:
  ```python
  from django.contrib import admin
  from .models import Item

  admin.site.register(Item)
  ```
- `http://127.0.0.1:8000/admin/` manzilida admin paneliga kiring va ma'lumot qo'shing.

## Xulosa
Bu darsda siz Django loyihasini noldan boshlab, DRF yordamida oddiy API yaratishni o'rgandingiz. Keyingi qadamlar sifatida autentifikatsiya, ruxsatlar va murakkab API funksiyalarini qo'shishingiz mumkin.