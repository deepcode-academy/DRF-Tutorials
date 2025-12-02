# 🧩 13-DARS: SIGNALLAR VA DRF BILAN ISHLASH

Bu darsda Django REST Framework (DRF) yordamida Django signallari bilan ishlashni bosqichma-bosqich o'rganamiz. Signallar ma'lumotlar bazasidagi o'zgarishlarga (masalan, yangi obyekt yaratilishi yoki o'chirilishi) avtomatik ravishda javob berish imkonini beradi. Har bir qadam tushunarli va faqat signallar va DRF integratsiyasiga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli, JWT autentifikatsiyasi, ruxsatlar va sahifalash asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan (shu jumladan `owner` maydoni).
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- `TaskViewSet` `myapp/views.py` faylida sozlangan (JWT autentifikatsiyasi, ruxsatlar, filtrlash va sahifalash bilan).
- `/tasks/` endpointi `myproject/urls.py` faylida sozlangan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. SIGNALLARNI TUSHUNISH
📌 **Django signallari** ma'lumotlar bazasidagi hodisalarga (masalan, obyekt yaratilishi, yangilanishi yoki o'chirilishi) avtomatik javob berish uchun ishlatiladi. DRFda signallar API orqali amalga oshirilgan o'zgarishlarga qo'shimcha logika qo'shishda foydalidir.
📌 Asosiy signallar:
- `post_save`: Obyekt saqlanganda ishga tushadi.
- `pre_save`: Obyekt saqlanishidan oldin ishga tushadi.
- `post_delete`: Obyekt o'chirilganda ishga tushadi.
📌 DRF bilan signallarni ishlatish API operatsiyalariga qo'shimcha funksionallik qo'shish uchun qulaydir (masalan, log yozish yoki bildirishnoma yuborish).

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
    owner = models.ForeignKey('auth.User', related_name='tasks', on_delete=models.CASCADE, null=True)

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
    owner = serializers.ReadOnlyField(source='owner.username')

    class Meta:
        model = Task
        fields = ['id', 'title', 'description', 'completed', 'created_at', 'status', 'owner']
        read_only_fields = ['id', 'created_at', 'owner']

    def get_status(self, obj):
        return "Bajarilgan" if obj.completed else "Bajarilmagan"

    def validate_title(self, value):
        if len(value) < 3:
            raise serializers.ValidationError("Sarlavha kamida 3 ta belgidan iborat bo'lishi kerak.")
        return value
```

## ✅ 4. SIGNALLARNI SOZLASH
📌 Signallar yordamida vazifa yaratilganda yoki o'chirilganda log yozishni amalga oshiramiz. `myapp/signals.py` faylini yarating va quyidagi kodni qo'shing:
```python
from django.db.models.signals import post_save, post_delete
from django.dispatch import receiver
from .models import Task
import logging

# Logger sozlash
logger = logging.getLogger(__name__)

@receiver(post_save, sender=Task)
def log_task_creation(sender, instance, created, **kwargs):
    if created:
        logger.info(f"Yangi vazifa yaratildi: {instance.title} (ID: {instance.id}, Owner: {instance.owner.username})")
    else:
        logger.info(f"Vazifa yangilandi: {instance.title} (ID: {instance.id}, Owner: {instance.owner.username})")

@receiver(post_delete, sender=Task)
def log_task_deletion(sender, instance, **kwargs):
    logger.info(f"Vazifa o'chirildi: {instance.title} (ID: {instance.id}, Owner: {instance.owner.username})")
```
📌 **Tushuntirish**:
- `post_save`: Vazifa yaratilganda yoki yangilanganda ishga tushadi.
- `post_delete`: Vazifa o'chirilganda ishga tushadi.
- `logger.info`: O'zgarishlarni log fayliga yozadi.

## ✅ 5. SIGNALLARNI ILovaga ULASH
📌 Signallarni ilovaga ulash uchun `myapp/apps.py` faylini yangilaymiz:
```python
from django.apps import AppConfig

class MyappConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'myapp'

    def ready(self):
        import myapp.signals  # Signallarni import qilish
```
📌 `myproject/settings.py` faylida `INSTALLED_APPS` da `myapp` ni aniq belgilang:
```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework.authtoken',
    'django_filters',
    'myapp.apps.MyappConfig',  # To'g'ri AppConfig
]
```

## ✅ 6. LOGGING SOZLAMALARINI QO'SHISH
📌 Loglarni ko'rish uchun `myproject/settings.py` fayliga logging sozlamalarini qo'shamiz:
```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': 'debug.log',
        },
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        'myapp': {
            'handlers': ['file', 'console'],
            'level': 'INFO',
            'propagate': True,
        },
    },
}
```
📌 **Tushuntirish**:
- Loglar `debug.log` fayliga yoziladi va konsolda ko'rinadi.
- `level: 'INFO'`: Faqat INFO va undan yuqori darajadagi xabarlar yoziladi.

## ✅ 7. VIEWSET NI TEKSHIRISH
📌 `myapp/views.py` faylida `TaskViewSet` oldingi darsdagidek qoladi:
```python
from rest_framework import viewsets
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.filters import SearchFilter, OrderingFilter
from .models import Task
from .serializers import TaskSerializer
from .permissions import IsOwnerOrReadOnly
from .pagination import CustomTaskPagination
from rest_framework.permissions import IsAuthenticated

class TaskViewSet(viewsets.ModelViewSet):
    queryset = Task.objects.all()
    serializer_class = TaskSerializer
    permission_classes = [IsAuthenticated, IsOwnerOrReadOnly]
    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
    filterset_fields = ['completed', 'owner']
    search_fields = ['title', 'description']
    ordering_fields = ['created_at', 'completed']
    ordering = ['created_at']
    pagination_class = CustomTaskPagination

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

## ✅ 8. URL SOZLASH
📌 `myproject/urls.py` faylida URL marshrutlari oldingi darsdagidek qoladi:
```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView
from myapp.views import TaskViewSet

router = DefaultRouter()
router.register(r'tasks', TaskViewSet)

urlpatterns = [
    path('', include(router.urls)),
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]
```

## ✅ 9. SIGNALLARNI SINAB KO'RISH
📌 Serverni ishga tushiring:
```bash
python manage.py runserver
```

### ❇️ **Foydalanuvchi va token yaratish**:
📌 Agar hali yaratilmagan bo'lsa, foydalanuvchi yarating:
```bash
python manage.py createsuperuser  # username: user1, password: pass123
```
📌 JWT token oling:
```bash
curl -X POST http://127.0.0.1:8000/api/token/ -H "Content-Type: application/json" -d '{"username": "user1", "password": "pass123"}'
```

### ❇️ **Vazifa yaratish va signal sinovi**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
📌 **Usul**: POST  
```bash
curl -X POST http://127.0.0.1:8000/tasks/ -H "Authorization: Bearer <access_token>" -H "Content-Type: application/json" -d '{"title": "Yangi vazifa", "description": "Test vazifa"}'
```
📌 Konsolda yoki `debug.log` faylida quyidagi logni ko'rasiz:
```
Yangi vazifa yaratildi: Yangi vazifa (ID: 1, Owner: user1)
```

### ❇️ **Vazifa yangilash va signal sinovi**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
📌 **Usul**: PATCH  
```bash
curl -X PATCH http://127.0.0.1:8000/tasks/1/ -H "Authorization: Bearer <access_token>" -H "Content-Type: application/json" -d '{"completed": true}'
```
📌 Log:
```
Vazifa yangilandi: Yangi vazifa (ID: 1, Owner: user1)
```

### ❇️ **Vazifa o'chirish va signal sinovi**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
📌 **Usul**: DELETE  
```bash
curl -X DELETE http://127.0.0.1:8000/tasks/1/ -H "Authorization: Bearer <access_token>"
```
📌 Log:
```
Vazifa o'chirildi: Yangi vazifa (ID: 1, Owner: user1)
```

## ✅ 10. ADMIN PANELI ORQALI SINOV (IXTIYORIY)
📌 Ma'lumotlarni qo'lda sinash uchun admin panelidan foydalaning:

### ❇️ **Admin foydalanuvchisini tekshirish**:
```bash
python manage.py createsuperuser
```

### ❇️ **Modelni admin panelida ro'yxatdan o'tkazish**:
📌 `myapp/admin.py` faylida:
```python
from django.contrib import admin
from .models import Task

admin.site.register(Task)
```

📌 `http://127.0.0.1:8000/admin/` manzilida vazifalarni qo'shing yoki tahrirlang va loglarni tekshiring.

## ✅ 11. SINOV UCHUN MASALALAR
📌 Signallarni sinash uchun quyidagi amallarni bajarib ko'ring:
1. `/tasks/` manzilida yangi vazifa qo'shing va `debug.log` da logni tekshiring.
2. `/tasks/1/` manzilida vazifani yangilang va logni ko'ring.
3. `/tasks/1/` manzilida vazifani o'chiring va logni tekshiring.
4. Admin panelida vazifa qo'shing va signal ishlaganligini tasdiqlang.

