# 🧩 3-DARS: CRUD OPERATSIYALARINI FUNCTION-BASED VIEWS BILAN AMALGA OSHIRISH

Bu darsda Django REST Framework (DRF) yordamida CRUD (Create, Read, Update, Delete) operatsiyalarini function-based views (FBV) yordamida bosqichma-bosqich amalga oshiramiz. Har bir qadam tushunarli va faqat CRUD operatsiyalariga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan.
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- Ma'lumotlar bazasi migratsiyalari qo'llanilgan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. CRUD OPERATSIYALARINI TUSHUNISH
📌 CRUD operatsiyalari quyidagilarni o'z ichiga oladi:
- **Create**: Yangi ma'lumot qo'shish.
- **Read**: Mavjud ma'lumotlarni o'qish (ro'yxat yoki bitta ob'ekt).
- **Update**: Mavjud ma'lumotlarni yangilash.
- **Delete**: Ma'lumotlarni o'chirish.

📌 Function-based views yordamida har bir operatsiyani alohida funksiya sifatida yozamiz.

## ✅ 3. MODEL VA SERIALIZERNI TEKSHIRISH
📌 Quyidagi sozlamalar oldingi darslarda amalga oshirilgan, lekin ularni qayta ko'rib chiqamiz:

### ❇️ **Model tekshirish**:
📌 `myapp/models.py` faylida `Task` modeli quyidagicha bo'lishi kerak:
```python
from django.db import models

class Task(models.Model):
    title = models.CharField(max_length=200)
    completed = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

### ❇️ **Serializer tekshirish**:
📌 `myapp/serializers.py` faylida `TaskSerializer` quyidagicha bo'lishi kerak:
```python
from rest_framework import serializers
from .models import Task

class TaskSerializer(serializers.ModelSerializer):
    class Meta:
        model = Task
        fields = ['id', 'title', 'completed', 'created_at']
```

## ✅ 4. FUNCTION-BASED VIEWS YARATISH
📌 `myapp/views.py` faylini yangilaymiz va CRUD operatsiyalari uchun function-based views yozamiz:
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
        serializer = TaskSerializer(task, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    elif request.method == 'DELETE':
        task.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

📌 **Tushuntirish**:
- `task_list`: GET (barcha vazifalarni o'qish) va POST (yangi vazifa qo'shish) so'rovlarini boshqaradi.
- `task_detail`: GET (bitta vazifani o'qish), PUT (vazifani yangilash) va DELETE (vazifani o'chirish) so'rovlarini boshqaradi.
- `@api_view` dekoratori DRFning API funksiyalarini yoqadi.

## ✅ 5. URL SOZLASH
📌 `myproject/urls.py` faylini yangilaymiz va function-based views uchun URL marshrutlarini qo'shamiz:
```python
from django.urls import path
from myapp.views import task_list, task_detail

urlpatterns = [
    path('tasks/', task_list, name='task-list'),
    path('tasks/<int:pk>/', task_detail, name='task-detail'),
]
```

📌 **Tushuntirish**:
- `/tasks/` manzili vazifalar ro'yxatini ko'rish va yangi vazifa qo'shish uchun.
- `/tasks/<pk>/` manzili bitta vazifani ko'rish, yangilash yoki o'chirish uchun.

## ✅ 6. CRUD OPERATSIYALARINI SINAB KO'RISH
📌 Serverni ishga tushiring:
```bash
python manage.py runserver
```

📌 Brauzerda yoki Postman kabi vositalar yordamida quyidagi manzillarni sinab ko'ring:

### ❇️ **Create (Yangi vazifa qo'shish)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
📌 **Usul**: POST  
📌 **Masalan** (Postman orqali):
```bash
curl -X POST http://127.0.0.1:8000/tasks/ -H "Content-Type: application/json" -d '{"title": "Yangi vazifa", "completed": false}'
```
📌 DRF interfeysi orqali ham yangi vazifa qo'shishingiz mumkin.

### ❇️ **Read (Ma'lumotlarni o'qish)**:
- **Barcha vazifalarni ko'rish**:  
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
  📌 **Usul**: GET  
  📌 Brauzerda bu manzilga o'tsangiz, barcha vazifalar JSON formatida chiqadi.
- **Bitta vazifani ko'rish**:  
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/` (1 - vazifa ID si)  
  📌 **Usul**: GET  
  📌 Masalan:
  ```bash
  curl http://127.0.0.1:8000/tasks/1/
  ```

### ❇️ **Update (Ma'lumotlarni yangilash)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
📌 **Usul**: PUT  
📌 **Masalan** (vazifani yangilash uchun):
```bash
curl -X PUT http://127.0.0.1:8000/tasks/1/ -H "Content-Type: application/json" -d '{"title": "Yangilangan vazifa", "completed": true}'
```

### ❇️ **Delete (Ma'lumotlarni o'chirish)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
📌 **Usul**: DELETE  
📌 **Masalan**:
```bash
curl -X DELETE http://127.0.0.1:8000/tasks/1/
```
📌 Bu 1-ID li vazifani o'chiradi.

## ✅ 7. ADMIN PANELI ORQALI SINOV (IXTIYORIY)
📌 Ma'lumotlarni qo'lda sinash uchun admin panelidan foydalaning:

### ❇️ **Admin foydalanuvchisini tekshirish**:
📌 Agar hali yaratilmagan bo'lsa:
```bash
python manage.py createsuperuser
```

### ❇️ **Modelni admin panelida ro'yxatdan o'tkazish**:
📌 `myapp/admin.py` faylida quyidagi kod bo'lishi kerak:
```python
from django.contrib import admin
from .models import Task

admin.site.register(Task)
```

📌 `http://127.0.0.1:8000/admin/` manzilida admin paneliga kiring va vazifalarni qo'shing, tahrirlang yoki o'chiring.

## ✅ 8. CRUD SINOVI UCHUN MASALALAR
📌 API funksionalligini sinash uchun quyidagi amallarni bajarib ko'ring:
1. Brauzer orqali `/tasks/` manzilida vazifalar ro'yxatini ko'ring.
2. Postman yoki DRF interfeysi yordamida yangi vazifa qo'shing (masalan, `title: "Test vazifa", completed: False`).
3. `/tasks/1/` manzilida bitta vazifani yangilang (masalan, `completed` ni `true` ga o'zgartiring).
4. `/tasks/1/` manzilidan foydalanib, 1-ID li vazifani o'chiring.
