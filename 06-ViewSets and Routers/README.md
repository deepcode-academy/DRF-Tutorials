# 🧩 6-DARS: VIEWSETS VA ROUTERS BILAN ISHLASH

Bu darsda Django REST Framework (DRF) yordamida ViewSets va Routers dan foydalanib, API endpointlarini samarali sozlashni bosqichma-bosqich o'rganamiz. ViewSets va Routers kod yozishni soddalashtiradi va URL marshrutlarini avtomatlashtiradi. Har bir qadam tushunarli va faqat ViewSets va Routers ga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan.
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- Ma'lumotlar bazasi migratsiyalari qo'llanilgan.
- `rest_framework` `INSTALLED_APPS` ro'yxatida qo'shilgan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. VIEWSETS VA ROUTERS NI TUSHUNISH
📌 **ViewSets**: DRFning ViewSets sinflari bir nechta API operatsiyalarini (masalan, CRUD) bitta sinf ichida birlashtiradi. Ular Generic API Views dan farqli o'laroq, barcha operatsiyalarni bitta sinfda boshqarish imkonini beradi. Eng keng tarqalgan ViewSet turi — `ModelViewSet`, u to'liq CRUD funksiyalarini qo'llab-quvvatlaydi.
📌 **Routers**: DRFning Routers komponenti ViewSets uchun URL marshrutlarini avtomatik ravishda yaratadi, bu esa URL sozlamalarini soddalashtiradi.

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

## ✅ 4. VIEWSET YARATISH
📌 `myapp/views.py` faylini yangilaymiz va `ModelViewSet` yordamida ViewSet sozlaymiz:
```python
from rest_framework import viewsets
from .models import Task
from .serializers import TaskSerializer

class TaskViewSet(viewsets.ModelViewSet):
    queryset = Task.objects.all()
    serializer_class = TaskSerializer
```
📌 **Tushuntirish**:
- `ModelViewSet`: Barcha CRUD operatsiyalarini (list, retrieve, create, update, delete) avtomatik qo'llab-quvvatlaydi.
- `queryset`: Qaysi ma'lumotlar bilan ishlashni belgilaydi (`Task` modelining barcha obyektlari).
- `serializer_class`: Ma'lumotlarni seriyalash va deserialash uchun ishlatiladigan serializer (`TaskSerializer`).

## ✅ 5. ROUTER BILAN URL SOZLASH
📌 DRFning `DefaultRouter` yordamida URL marshrutlarini avtomatik sozlaymiz. `myproject/urls.py` faylini yangilaymiz:
```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from myapp.views import TaskViewSet

router = DefaultRouter()
router.register(r'tasks', TaskViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```
📌 **Tushuntirish**:
- `DefaultRouter`: `/tasks/` va `/tasks/<pk>/` kabi marshrutlarni avtomatik yaratadi.
- `router.register(r'tasks', TaskViewSet)`: `TaskViewSet` ni `tasks` prefiksi bilan bog'laydi.
- `include(router.urls)`: Router tomonidan yaratilgan barcha URL larni loyihaga qo'shadi.

📌 Router quyidagi endpointlarni yaratadi:
- `GET /tasks/`: Barcha vazifalarni ro'yxatlash.
- `POST /tasks/`: Yangi vazifa qo'shish.
- `GET /tasks/<pk>/`: Bitta vazifani ko'rish.
- `PUT/PATCH /tasks/<pk>/`: Vazifani yangilash.
- `DELETE /tasks/<pk>/`: Vazifani o'chirish.

## ✅ 6. VIEWSETS VA ROUTERS NI SINAB KO'RISH
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
📌 Agar `title` 3 belgidan kam bo'lsa, `TaskSerializer` xato qaytaradi.

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
📌 **Usul**: PATCH  
📌 **Masalan** (faqat `completed` ni yangilash):
```bash
curl -X PATCH http://127.0.0.1:8000/tasks/1/ -H "Content-Type: application/json" -d '{"completed": true}'
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
📌 ViewSets va Routers funksionalligini sinash uchun quyidagi amallarni bajarib ko'ring:
1. `/tasks/` manzilida yangi vazifa qo'shing va `title` 3 belgidan kam bo'lsa, xato oling.
2. `/tasks/` manziliga GET so'rov yuboring va `status` maydonini tekshiring.
3. `/tasks/1/` manzilida vazifani yangilang, faqat `description` ni o'zgartiring.
4. `/tasks/1/` manzilidan foydalanib, 1-ID li vazifani o'chiring.

