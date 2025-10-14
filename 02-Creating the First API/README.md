# 🌐 DJANGO REST FRAMEWORK ASOSLARI

# 🧩 2-DARS: BIRINCHI API NI YARATISH

Bu darsda Django REST Framework (DRF) yordamida birinchi API ni bosqichma-bosqich yaratamiz. Har bir qadam tushunarli va faqat API yaratishga qaratilgan bo'ladi. Oldingi darsda sozlangan `myproject` loyihasi va `myapp` ilovasi asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject` va `myapp`). Agar hali sozlamagan bo'lsangiz, avvalgi darsga qayting. Quyidagi sozlamalar mavjud bo'lishi kerak:
- `myapp` ilovasi `INSTALLED_APPS` ro'yxatida.
- `rest_framework` ham `INSTALLED_APPS` ga qo'shilgan.
- Virtual muhit faol va Django bilan DRF o'rnatilgan.

## ✅ 2. MODELNI YARATISH
📌 API ma'lumotlar bilan ishlashi uchun avval model yaratamiz. `myapp/models.py` faylida quyidagi oddiy modelni qo'shamiz:
```python
from django.db import models

class Task(models.Model):
    title = models.CharField(max_length=200)
    completed = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

📌 Bu model vazifalar ro'yxatini saqlash uchun ishlatiladi (masalan, todo-list).

### ❇️ **Migratsiyalarni amalga oshirish**:
📌 Modelni ma'lumotlar bazasiga qo'shish uchun migratsiyalarni yarating va qo'llang:
```bash
python manage.py makemigrations
python manage.py migrate
```

## ✅ 3. SERIALIZER YARATISH
📌 Serializer ma'lumotlarni JSON formatiga aylantiradi va API orqali ishlatish uchun tayyorlaydi. `myapp/serializers.py` faylini yarating va quyidagi kodni qo'shing:
```python
from rest_framework import serializers
from .models import Task

class TaskSerializer(serializers.ModelSerializer):
    class Meta:
        model = Task
        fields = ['id', 'title', 'completed', 'created_at']
```

📌 `fields` ro'yxatida API orqali ko'rsatiladigan maydonlarni belgilaymiz.

## ✅ 4. VIEW YARATISH
📌 API funksiyalarini boshqarish uchun view yaratamiz. `myapp/views.py` faylida quyidagi kodni qo'shing:
```python
from rest_framework import viewsets
from .models import Task
from .serializers import TaskSerializer

class TaskViewSet(viewsets.ModelViewSet):
    queryset = Task.objects.all()
    serializer_class = TaskSerializer
```

📌 `ModelViewSet` DRFning tayyor sinfi bo'lib, CRUD (Create, Read, Update, Delete) amallarini avtomatik boshqaradi.

## ✅ 5. URL SOZLASH
📌 API uchun URL marshrutlarini sozlaymiz. `myproject/urls.py` faylini yangilang:
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

📌 `router.register` orqali `tasks` endpointi yaratiladi. Bu `/tasks/` manzilida API ishlaydi.

## ✅ 6. API NI SINAB KO'RISH
📌 Serverni ishga tushiring:
```bash
python manage.py runserver
```

📌 Brauzerda `http://127.0.0.1:8000/tasks/` manziliga o'ting. DRFning interfeysi orqali quyidagi amallarni sinab ko'rishingiz mumkin:
- **GET**: Barcha vazifalarni ko'rish.
- **POST**: Yangi vazifa qo'shish.
- **PUT/PATCH**: Mavjud vazifani tahrirlash.
- **DELETE**: Vazifani o'chirish.

## ✅ 7. MA'LUMOT QO'SHISH (IXTIYORIY)
📌 Django admin paneli orqali sinov ma'lumotlari qo'shish uchun:

### ❇️ **Admin foydalanuvchisini yaratish**:
```bash
python manage.py createsuperuser
```

### ❇️ **Modelni admin panelida ro'yxatdan o'tkazish**:
📌 `myapp/admin.py` faylida quyidagi kodni qo'shing:
```python
from django.contrib import admin
from .models import Task

admin.site.register(Task)
```

📌 `http://127.0.0.1:8000/admin/` manzilida admin paneliga kiring va vazifalar qo'shing.

## ✅ 8. API NI TEST QILISH UCHUN MASALALAR
📌 API funksionalligini sinash uchun quyidagi amallarni bajarib ko'ring:
1. Brauzer orqali `/tasks/` manzilida vazifalar ro'yxatini ko'ring.
2. DRF interfeysi yordamida yangi vazifa qo'shing (masalan, `title: "Birinchi vazifa", completed: False`).
3. `curl` yoki Postman yordamida quyidagi so'rovni yuboring:
   ```bash
   curl -X POST http://127.0.0.1:8000/tasks/ -d "title=Test vazifa&completed=true"
   ```

## ✅ XULOSA
Bu darsda siz DRF yordamida birinchi API ni yaratdingiz. `Task` modeli asosida vazifalar ro'yxatini boshqarish uchun API endpoint sozladik. Keyingi qadamlar sifatida autentifikatsiya, ruxsatlar va filtrlash kabi qo'shimcha funksiyalarni qo'shishingiz mumkin.