# 🌐 DJANGO REST FRAMEWORK ASOSLARI

# 🧩 5-DARS: GENERIC API VIEWS BILAN ISHLASH

Bu darsda Django REST Framework (DRF) yordamida Generic API Views yordamida CRUD (Create, Read, Update, Delete) operatsiyalarini bosqichma-bosqich amalga oshiramiz. Generic API Views kod yozishni soddalashtiradi va takrorlanadigan logikani kamaytiradi. Har bir qadam tushunarli va faqat Generic API Views ga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan.
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- Ma'lumotlar bazasi migratsiyalari qo'llanilgan.
- `rest_framework` `INSTALLED_APPS` ro'yxatida qo'shilgan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. GENERIC API VIEWS NI TUSHUNISH
📌 DRFning Generic API Views sinflari umumiy API operatsiyalari uchun tayyor yechimlarni taqdim etadi. Ular function-based views (FBV) ga nisbatan kamroq kod yozishni talab qiladi va `ModelViewSet` dan farqli o'laroq, ko'proq moslashuvchanlik beradi. Asosiy generic sinflar:
- `ListCreateAPIView`: Ro'yxatni ko'rish (GET) va yangi obyekt qo'shish (POST).
- `RetrieveUpdateDestroyAPIView`: Bitta obyektni ko'rish (GET), yangilash (PUT/PATCH) va o'chirish (DELETE).

## ✅ 3. MODEL VA SERIALIZERNI TEKSHIRISH
📌 Quyidagi sozlamalar oldingi darslarda amalga oshirilgan, lekin ularni qayta ko'rib chiqamiz:

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

### ❇️ **Serializer tekshirish**:
📌 `myapp/serializers.py` faylida `TaskSerializer` quyidagicha bo'lishi kerak:
```python
from rest_framework import serializers
from .models import Task

class TaskSerializer(serializers.ModelSerializer):
    status = serializers.SerializerMethodField()

    class Meta:
        model = Task
        fields = ['id', 'title', 'description', 'completed', 'created_at', 'status']
        read_only_fields = ['id', 'created_at']

    def get_status(self, obj):
        return "Bajarilgan" if obj.completed else "Bajarilmagan"

    def validate_title(self, value):
        if len(value) < 3:
            raise serializers.ValidationError("Sarlavha kamida 3 ta belgidan iborat bo'lishi kerak.")
        return value
```

## ✅ 4. GENERIC API VIEWS YARATISH
📌 `myapp/views.py` faylini yangilaymiz va Generic API Views yordamida CRUD operatsiyalarini sozlaymiz:
```python
from rest_framework import generics
from .models import Task
from .serializers import TaskSerializer

class TaskListCreateView(generics.ListCreateAPIView):
    queryset = Task.objects.all()
    serializer_class = TaskSerializer

class TaskRetrieveUpdateDestroyView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Task.objects.all()
    serializer_class = TaskSerializer
```
📌 **Tushuntirish**:
- `TaskListCreateView`: `ListCreateAPIView` sinfidan foydalanadi, bu GET (ro'yxatni ko'rish) va POST (yangi vazifa qo'shish) so'rovlarini boshqaradi.
- `TaskRetrieveUpdateDestroyView`: `RetrieveUpdateDestroyAPIView` sinfidan foydalanadi, bu GET (bitta vazifani ko'rish), PUT/PATCH (yangilash) va DELETE (o'chirish) so'rovlarini boshqaradi.
- `queryset`: Qaysi ma'lumotlar bilan ishlashni belgilaydi.
- `serializer_class`: Ma'lumotlarni seriyalash va deserialash uchun ishlatiladigan serializer.

## ✅ 5. URL SOZLASH
📌 `myproject/urls.py` faylini yangilaymiz va Generic API Views uchun URL marshrutlarini qo'shamiz:
```python
from django.urls import path
from myapp.views import TaskListCreateView, TaskRetrieveUpdateDestroyView

urlpatterns = [
    path('tasks/', TaskListCreateView.as_view(), name='task-list-create'),
    path('tasks/<int:pk>/', TaskRetrieveUpdateDestroyView.as_view(), name='task-detail'),
]
```
📌 **Tushuntirish**:
- `/tasks/` manzili vazifalar ro'yxatini ko'rish va yangi vazifa qo'shish uchun.
- `/tasks/<pk>/` manzili bitta vazifani ko'rish, yangilash yoki o'chirish uchun.
- `as_view()` metodi sinfni API so'rovlariga moslashtiradi.

## ✅ 6. GENERIC API VIEWS NI SINAB KO'RISH
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
curl -X POST http://127.0.0.1:8000/tasks/ -H "Content-Type: application/json" -d '{"title": "Yangi vazifa", "description": "Bu test vazifa", "completed": false}'
```
📌 Agar `title` 3 belgidan kam bo'lsa, `ModelSerializer` xato qaytaradi.

### ❇️ **Read (Ma'lumotlarni o'qish)**:
- **Barcha vazifalarni ko'rish**:  
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
  📌 **Usul**: GET  
  📌 JSON formatida barcha vazifalar, shu jumladan `status` maydoni ko'rinadi.
- **Bitta vazifani ko'rish**:  
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
  📌 **Usul**: GET  
  📌 Masalan:
  ```bash
  curl http://127.0.0.1:8000/tasks/1/
  ```

### ❇️ **Update (Ma'lumotlarni yangilash)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
📌 **Usul**: PUT yoki PATCH  
📌 **Masalan** (faqat `description` ni yangilash):
```bash
curl -X PATCH http://127.0.0.1:8000/tasks/1/ -H "Content-Type: application/json" -d '{"description": "Yangilangan tavsif"}'
```

### ❇️ **Delete (Ma'lumotlarni o'chirish)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
📌 **Usul**: DELETE  
📌 **Masalan**:
```bash
curl -X DELETE http://127.0.0.1:8000/tasks/1/
```

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

## ✅ 8. SINOV UCHUN MASALALAR
📌 Generic API Views funksionalligini sinash uchun quyidagi amallarni bajarib ko'ring:
1. `/tasks/` manzilida yangi vazifa qo'shing va `title` 3 belgidan kam bo'lsa, xato oling.
2. `/tasks/` manziliga GET so'rov yuboring va `status` maydonini tekshiring.
3. `/tasks/1/` manzilida vazifani yangilang, faqat `completed` ni `true` ga o'zgartiring.
4. `/tasks/1/` manzilidan foydalanib, 1-ID li vazifani o'chiring.
