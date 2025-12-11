# 🧩 4-DARS: MODELSERIALIZER BILAN ISHLASH

Bu darsda Django REST Framework (DRF) yordamida `ModelSerializer` sinfidan foydalanib, ma'lumotlarni seriyalash va deserialash jarayonlarini bosqichma-bosqich o'rganamiz. Har bir qadam tushunarli va faqat `ModelSerializer` ga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan.
- Ma'lumotlar bazasi migratsiyalari qo'llanilgan.
- `rest_framework` `INSTALLED_APPS` ro'yxatida qo'shilgan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. MODELSERIALIZER NI TUSHUNISH
📌 `ModelSerializer` DRFning asosiy komponentlaridan biri bo'lib, Django modellarini JSON formatiga aylantirish (seriyalash) va JSON dan model obyektlariga aylantirish (deserialash) jarayonlarini soddalashtiradi. U quyidagi afzalliklarni beradi:
- Model maydonlarini avtomatik aniqlaydi.
- Validatsiya qoidalarini modelga asoslangan holda yaratadi.
- CRUD operatsiyalari uchun tayyor metodlarni taqdim etadi.

## ✅ 3. MODEL VA SERIALIZER SOZLASH
📌 Quyidagi sozlamalar oldingi darslarda amalga oshirilgan, lekin ularni qayta ko'rib chiqamiz va `ModelSerializer` ni kengaytiramiz.

### ❇️ **Model tekshirish**:
📌 `myapp/models.py` faylida `Task` modeli quyidagicha bo'lishi kerak:
```python
from django.db import models

class Task(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True, null=True)
    completed = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```
📌 Yangilik sifatida `description` maydonini qo'shdik, bu ixtiyoriy maydon (`blank=True, null=True`).

### ❇️ **ModelSerializer yaratish**:
📌 `myapp/serializers.py` faylini yangilaymiz va `ModelSerializer` ni kengaytirib, qo'shimcha validatsiya va maydonlarni sozlaymiz:
```python
from rest_framework import serializers
from .models import Task

class TaskSerializer(serializers.ModelSerializer):
    class Meta:
        model = Task
        fields = ['id', 'title', 'description', 'completed', 'created_at']
        read_only_fields = ['id', 'created_at']

    def validate_title(self, value):
        if len(value) < 3:
            raise serializers.ValidationError("Sarlavha kamida 3 ta belgidan iborat bo'lishi kerak.")
        return value
```
📌 **Tushuntirish**:
- `fields`: API orqali ko'rsatiladigan maydonlar ro'yxati.
- `read_only_fields`: Faqat o'qish uchun maydonlar (masalan, `id` va `created_at` foydalanuvchi tomonidan o'zgartirilmaydi).
- `validate_title`: Sarlavha uzunligini tekshiruvchi maxsus validatsiya qoidasi.

## ✅ 4. FUNCTION-BASED VIEWS BILAN MODELSERIALIZER ISHLATISH
📌 Oldingi darsda function-based views (FBV) yordamida CRUD operatsiyalarini sozlagan edik. `ModelSerializer` dan foydalanish uchun `myapp/views.py` faylini qayta ko'rib chiqamiz:
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from .models import Task
from .serializers import TaskSerializer

@api_view(['GET', 'POST'])
def task_list(request):
    if request.method == 'GET':
        tasks = Task.objects.all()
        serializer = TaskSerializer(tasks, many=True)
        return Response(serializer.data)
    
    elif request.method == 'POST':
        serializer = TaskSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET', 'PUT', 'DELETE'])
def task_detail(request, pk):
    try:
        task = Task.objects.get(pk=pk)
    except Task.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)
    
    if request.method == 'GET':
        serializer = TaskSerializer(task)
        return Response(serializer.data)
    
    elif request.method == 'PUT':
        serializer = TaskSerializer(task, data=request.data, partial=True)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    elif request.method == 'DELETE':
        task.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```
📌 **Tushuntirish**:
- `TaskSerializer` barcha CRUD operatsiyalarida ishlatiladi.
- `partial=True` (PUT so'rovida) qisman yangilash imkonini beradi, ya'ni faqat o'zgartirilgan maydonlar yangilanadi.
- `serializer.is_valid()` `ModelSerializer` ning validatsiya qoidalarini (masalan, `validate_title`) tekshiradi.

## ✅ 5. URL SOZLASH
📌 `myproject/urls.py` faylida URL marshrutlari oldingi darsdagidek qoladi:
```python
from django.urls import path
from myapp.views import task_list, task_detail

urlpatterns = [
    path('tasks/', task_list, name='task-list'),
    path('tasks/<int:pk>/', task_detail, name='task-detail'),
]
```

## ✅ 6. MODELSERIALIZER NI SINAB KO'RISH
📌 Serverni ishga tushiring:
```bash
python manage.py runserver
```

📌 Brauzerda yoki Postman kabi vositalar yordamida quyidagi manzillarni sinab ko'ring:

### ❇️ **Create (Yangi vazifa qo'shish)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
📌 **Usul**: POST  
📌 **Masalan**:
```bash
curl -X POST http://127.0.0.1:8000/tasks/ -H "Content-Type: application/json" -d '{"title": "Yangi vazifa", "description": "Bu test vazifa", "completed": false}'
```
📌 Agar `title` 3 belgidan kam bo'lsa, `ModelSerializer` xato qaytaradi.

### ❇️ **Read (Ma'lumotlarni o'qish)**:
- **Barcha vazifalarni ko'rish**:  
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
  📌 **Usul**: GET  
- **Bitta vazifani ko'rish**:  
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
  📌 **Usul**: GET  

### ❇️ **Update (Ma'lumotlarni yangilash)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
📌 **Usul**: PUT  
📌 **Masalan** (faqat `description` ni yangilash):
```bash
curl -X PUT http://127.0.0.1:8000/tasks/1/ -H "Content-Type: application/json" -d '{"description": "Yangilangan tavsif"}'
```

### ❇️ **Delete (Ma'lumotlarni o'chirish)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
📌 **Usul**: DELETE  
📌 **Masalan**:
```bash
curl -X DELETE http://127.0.0.1:8000/tasks/1/
```

## ✅ 7. MODELSERIALIZER NI KENGAYTIRISH
📌 `ModelSerializer` ni yanada moslashtirish uchun qo'shimcha funksiyalar qo'shamiz. `myapp/serializers.py` faylini yangilaymiz:
```python
from rest_framework import serializers
from .models import Task

class TaskSerializer(serializers.ModelSerializer):
    class Meta:
        model = Task
        fields = ['id', 'title', 'description', 'completed', 'created_at', 'status']
        read_only_fields = ['id', 'created_at']

    status = serializers.SerializerMethodField()

    def get_status(self, obj):
        return "Bajarilgan" if obj.completed else "Bajarilmagan"

    def validate_title(self, value):
        if len(value) < 3:
            raise serializers.ValidationError("Sarlavha kamida 3 ta belgidan iborat bo'lishi kerak.")
        return value
```
📌 **Tushuntirish**:
- `status`: `SerializerMethodField` yordamida `completed` maydoniga asoslangan virtual maydon qo'shildi.
- `get_status`: `completed` holatiga qarab "Bajarilgan" yoki "Bajarilmagan" qaytaradi.

📌 Yangi serializer ni sinash uchun `/tasks/` manziliga GET so'rov yuboring. Har bir vazifada `status` maydoni paydo bo'ladi.

## ✅ 8. SINOV UCHUN MASALALAR
📌 `ModelSerializer` funksionalligini sinash uchun quyidagi amallarni bajarib ko'ring:
1. `/tasks/` manzilida yangi vazifa qo'shing va `title` 3 belgidan kam bo'lsa, xato oling.
2. `/tasks/1/` manzilida vazifani yangilang, faqat `description` ni o'zgartiring.
3. `/tasks/` manziliga GET so'rov yuboring va `status` maydonini tekshiring.
4. Admin panelida bir nechta vazifa qo'shing va API orqali ularni ko'ring.

## ✅ 9. ADMIN PANELI ORQALI SINOV (IXTIYORIY)
📌 Ma'lumotlarni qo'lda sinash uchun admin panelidan foydalaning:
### ❇️ **Admin foydalanuvchisini tekshirish**:
```bash
python manage.py createsuperuser
```
### ❇️ **Modelni ro'yxatdan o'tkazish**:
📌 `myapp/admin.py` faylida:
```python
from django.contrib import admin
from .models import Task

admin.site.register(Task)
```
📌 `http://127.0.0.1:8000/admin/` manzilida vazifalarni qo'shing yoki tahrirlang.

