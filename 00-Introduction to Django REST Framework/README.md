# ✳️ DJANGO REST FRAMEWORK ASOSLARI


# 🧩 INTRODUCTION TO DJANGO REST FRAMEWORK

## ✅ DJANGO REST FRAMEWORK (DRF) NIMA?

📌 DRF – bu Django REST Framework degan so‘zlarning qisqartmasi. Bu Django frameworki ustiga qo‘shimcha kutubxona bo‘lib, u orqali biz API (Application Programming Interface) yaratamiz.

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

- `GET` — ma'lumot olish
- `POST` — ma'lumot yaratish
- `PUT/PATCH` — ma'lumotni yangilash
- `DELETE` — ma'lumotni o‘chirish

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

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('base.urls')),
]
```

## ✅ DRFDA ODDIY API YARATISH

### 1. Model

```python
# models.py

from django.db import models

class Student(models.Model):
    name = models.CharField(max_length=50)
    age = models.IntegerField()
    email = models.EmailField()

    def __str__(self):
        return self.name
```

📌 **DRF da serializer** – bu Django modelidagi ma’lumotlarni JSON, XML yoki boshqa formatlarga o‘tkazish uchun ishlatiladigan vosita. Shu bilan birga, u ma’lumotlarni tekshirish (validation) va yaratish/yangilash (create/update) imkonini beradi. 

```python
# serializers.py

from rest_framework import serializers
from .models import Student

class StudentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Student
        fields = ['id', 'name', 'age', 'email']
```

```python
# views.py

from rest_framework.decorators import api_view
from rest_framework.response import Response
from .models import Student
from .serializers import StudentSerializer

@api_view(['GET'])
def student_list(request):
    students = Student.objects.all()          # Barcha studentlarni olish
    serializer = StudentSerializer(students, many=True)  # JSONga aylantirish
    return Response(serializer.data)          # Foydalanuvchiga yuborish
```

- many=True → ko‘p obyektlarni JSON formatga aylantirish uchun
- serializer.data → JSON ma’lumotlar

```python
# urls.py

from django.urls import path
from .views import student_list

urlpatterns = [
    path('students/', student_list),
]
```

📌 POST so‘rov bilan yangi student qo‘shish

```python
@api_view(['GET', 'POST'])
def student_list(request):
    if request.method == 'GET':
        students = Student.objects.all()
        serializer = StudentSerializer(students, many=True)
        return Response(serializer.data)
    
    elif request.method == 'POST':
        serializer = StudentSerializer(data=request.data)  # Foydalanuvchidan ma'lumot olish
        if serializer.is_valid():                           # Tekshirish
            serializer.save()                               # Bazaga saqlash
            return Response(serializer.data, status=201)
        return Response(serializer.errors, status=400)     # Xato bo‘lsa
```
