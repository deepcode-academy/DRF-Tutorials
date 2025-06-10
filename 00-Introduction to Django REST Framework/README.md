# ✳️ DJANGO REST FRAMEWORK ASOSLARI


# 🧩 0-DARS INTRODUCTION TO DJANGO REST FRAMEWORK

## ✅ DJANGO REST FRAMEWORK (DRF) NIMA?

📌 Django REST Framework (DRF) — bu Django asosida qurilgan kuchli va moslashuvchan kutubxona bo‘lib, RESTful API larni yaratish uchun xizmat qiladi.

📌 DRF yordamida biz backend (ma’lumotlar bazasi bilan ishlovchi qism) bilan frontend yoki mobil ilovalar o‘rtasida JSON formatidagi muloqotni tashkil qilamiz.

📌 Bu kutubxona nafaqat CRUD amallarni amalga oshirish, balki authentication, permission, pagination, throttling, filtering, versioning kabi ilg‘or funksiyalarni ham osonlashtiradi.


## ✅ DRF NIMA UCHUN KERAK?

📌 Django odatda veb sahifalar (HTML) bilan ishlashga moslashgan. Ammo ko‘plab zamonaviy ilovalar (masalan, mobil ilovalar, frontend (React, Vue)) uchun API (Application Programming Interface) talab qilinadi.


📌 DRF aynan shunday holatlarda:

- 🔁 Django ma'lumotlarini JSON formatida frontend/mobil ilovaga uzatishda
- ✅ CRUD (Create, Read, Update, Delete) amallarini API orqali bajarishda
- 🔐 Token/Session asosida authentication va permission larni tashkil qilishda
- 🧪 API larni tez, xavfsiz va testga yaroqli qilishda juda foydalidir.

## ✅ REST API NIMA?

📌 REST (Representational State Transfer) — bu internet orqali resurslar (odatda ma’lumotlar) bilan ishlash usuli.

📌 REST API bu:

- **GET** — ma'lumot olish
- **POST** — ma'lumot yaratish
- **PUT/PATCH** — ma'lumotni yangilash
- **DELETE** — ma'lumotni o‘chirish

## ✅ NEGA DRF KERAK?

📌 Faraz qilaylik, sizda Kitob modeli bor:

```python
# models.py

# Django framework'dan models moduli import qilinmoqda, bu model yaratish uchun kerak bo'ladi
from django.db import models

# Kitob nomli model (jadval) yaratilmoqda, u models.Model dan meros oladi
class Book(models.Model):
    # 'name' - kitob nomini saqlash uchun CharField, maksimal uzunligi 100 ta belgidan iborat
    name = models.CharField(max_length=100)
    
    # 'author' - kitob muallifining ismi, CharField, maksimal uzunligi 100 ta belgidan iborat
    author = models.CharField(max_length=100)
    
    # 'date' - kitob chop etilgan sana, DateField tipida saqlanadi (faqat sana, vaqt emas)
    date = models.DateField()
```

📌 Agar siz bu modelni React frontend yoki mobil ilova bilan ulamoqchi bo‘lsangiz, sizga API kerak bo‘ladi. Django oddiy holatda bunday JSON API bermaydi. Bu yerda DRF yordamga keladi.

## ✅ DRF O‘RNATISH

```shell
pip install djangorestframework  
```

📌 `settings.py` faylga DRF ni qo‘shing:

```python
INSTALLED_APPS = [
    ...
    
    # Django REST Framework (DRF) — bu Django uchun API yaratish imkonini beradigan kuchli kutubxona.
    # Uni INSTALLED_APPS ga qo‘shish orqali DRF ning komponentlari (serializers, views, permissions va h.k.) loyihada ishlay oladi.
    # Masalan, DRF yordamida JSON formatda API endpointlar, CRUD amallarini bajaruvchi class-based yoki function-based viewlar yozish mumkin.
    # Bu qatorda 'rest_framework' yozilishi orqali DRF Django tomonidan tan olinadi va ishga tushiriladi.
    'rest_framework',
]
```

## ✅ DRF DA ODDIY API YARATISH

### 1. Model

```python
# models.py

# Book nomli model yaratilyapti, bu model bazada 'book' jadvalini ifodalaydi
class Book(models.Model):
    # 'name' maydoni — bu kitobning nomini saqlaydi.
    # CharField matnli qiymatlarni saqlash uchun ishlatiladi.
    # max_length=100 — bu maydonga kiritiladigan matn eng ko‘pi bilan 100 belgidan iborat bo‘lishi kerak.
    name = models.CharField(max_length=100)
    
    # 'author' maydoni — bu kitob muallifining ismini saqlaydi.
    # Yana CharField bo‘lib, 100 belgigacha matn qabul qiladi.
    author = models.CharField(max_length=100)
```