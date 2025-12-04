# 🧩 14-DARS: FON VAZIFALARI VA CELERY BILAN ISHLASH

Bu darsda Django REST Framework (DRF) yordamida fon vazifalarini (background tasks) va Celery bilan ishlashni bosqichma-bosqich o'rganamiz. Celery uzoq davom etadigan yoki resurs talab qiladigan vazifalarni asinxron tarzda bajarish uchun ishlatiladi. Har bir qadam tushunarli va faqat fon vazifalari va Celery ga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli, JWT autentifikatsiyasi, ruxsatlar va signallar asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan (shu jumladan `owner` maydoni).
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- `TaskViewSet` `myapp/views.py` faylida sozlangan (JWT autentifikatsiyasi, ruxsatlar, filtrlash va sahifalash bilan).
- Signallar `myapp/signals.py` faylida sozlangan.
- `/tasks/` endpointi `myproject/urls.py` faylida sozlangan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. CELERY VA FON VAZIFALARINI TUSHUNISH
📌 **Celery** — Python loyihalarida asinxron vazifalarni boshqarish uchun kuchli vosita. U uzoq vaqt talab qiladigan vazifalarni (masalan, log yozish, email yuborish) asosiy ilova jarayonidan ajratib, fon rejimida bajaradi.
📌 **Fon vazifalari**:
- API so'rovlarining tezligini oshiradi.
- Serverni ortiqcha yukdan himoya qiladi.
- Vazifalarni parallel ravishda bajarish imkonini beradi.
📌 Celery ishlatish uchun **Redis** yoki **RabbitMQ** kabi xabar brokeri talab qilinadi. Ushbu darsda Redis dan foydalanamiz.

## ✅ 3. CELERY VA REDIS NI O'RNATISH
📌 Quyidagi paketlarni o'rnatamiz:
```bash
pip install celery redis
```
📌 Redis ni o'rnatish (masalan, Ubuntu uchun):
```bash
sudo apt-get install redis-server
```
📌 Redis serverini ishga tushiring va uning ishlayotganligini tekshiring:
```bash
redis-server
redis-cli ping  # Javob: PONG
```

## ✅ 4. CELERY SOZLAMALARINI QO'SHISH
📌 `myproject` papkasida `celery.py` faylini yarating va quyidagi kodni qo'shing:
```python
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

app = Celery('myproject')

app.config_from_object('django.conf:settings', namespace='CELERY')

app.autodiscover_tasks()
```
📌 **Tushuntirish**:
- `Celery('myproject')`: Celery ilovasini yaratadi.
- `config_from_object`: Django sozlamalaridan Celery konfiguratsiyasini oladi.
- `autodiscover_tasks`: Ilovadagi vazifalarni avtomatik topadi.

📌 `myproject/__init__.py` faylini yangilang:
```python
from .celery import app as celery_app

__all__ = ('celery_app',)
```

## ✅ 5. LOYIHA SOZLAMALARINI YANGILASH
📌 `myproject/settings.py` faylida Celery va Redis sozlamalarini qo'shamiz:
```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework.authtoken',
    'django_filters',
    'myapp.apps.MyappConfig',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '10/hour',
        'user': '20/hour',
    },
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 5,
}

# Celery sozlamalari
CELERY_BROKER_URL = 'redis://localhost:6379/0'  # Redis broker manzili
CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'  # Natijalarni saqlash uchun
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_TIMEZONE = 'Asia/Tashkent'  # O'zbekiston vaqt zonasi
```
📌 **Tushuntirish**:
- `CELERY_BROKER_URL`: Redis server manzili (standart port 6379).
- `CELERY_TIMEZONE`: Vaqt zonasi sifatida `Asia/Tashkent` ishlatiladi.

## ✅ 6. FON VAZIFASINI YARATISH
📌 `myapp/tasks.py` faylini yarating va vazifa yaratilganda log yozish uchun fon vazifasini qo'shing:
```python
from celery import shared_task
import logging

logger = logging.getLogger(__name__)

@shared_task
def log_task_creation(task_id, task_title, owner_username):
    logger.info(f"[Celery] Yangi vazifa yaratildi: {task_title} (ID: {task_id}, Owner: {owner_username})")
```
📌 **Tushuntirish**:
- `@shared_task`: Celery vazifasini belgilaydi.
- `log_task_creation`: Vazifa yaratilganda log yozadi.

## ✅ 7. SIGNALLARNI CELERY BILAN INTEGRATSIYA QILISH
📌 `myapp/signals.py` faylini yangilaymiz va vazifa yaratilganda Celery vazifasini chaqiramiz:
```python
from django.db.models.signals import post_save, post_delete
from django.dispatch import receiver
from .models import Task
from .tasks import log_task_creation
import logging

logger = logging.getLogger(__name__)

@receiver(post_save, sender=Task)
def handle_task_creation(sender, instance, created, **kwargs):
    if created:
        # Celery vazifasini fon rejimida ishga tushirish
        log_task_creation.delay(instance.id, instance.title, instance.owner.username)
    else:
        logger.info(f"Vazifa yangilandi: {instance.title} (ID: {instance.id}, Owner: {instance.owner.username})")

@receiver(post_delete, sender=Task)
def log_task_deletion(sender, instance, **kwargs):
    logger.info(f"Vazifa o'chirildi: {instance.title} (ID: {instance.id}, Owner: {instance.owner.username})")
```
📌 **Tushuntirish**:
- `log_task_creation.delay`: Celery vazifasini asinxron tarzda ishga tushiradi.
- `post_delete` signali o'zgarmagan holda qoldi.

## ✅ 8. CELERY ISHGA TUSHIRISH
📌 Celery worker ni ishga tushirish uchun yangi terminal oching va loyiha papkasida quyidagi buyruqni bajaring:
```bash
celery -A myproject worker --loglevel=info
```
📌 **Eslatma**: Redis serveri ishlayotgan bo'lishi kerak.

## ✅ 9. VIEWSET NI TEKSHIRISH
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

## ✅ 10. URL SOZLASH
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

## ✅ 11. CELERY VAZIFALARINI SINAB KO'RISH
📌 Serverni ishga tushiring:
```bash
python manage.py runserver
```
📌 Celery worker ni alohida terminalda ishga tushiring:
```bash
celery -A myproject worker --loglevel=info
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

### ❇️ **Vazifa yaratish va Celery sinovi**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
📌 **Usul**: POST  
```bash
curl -X POST http://127.0.0.1:8000/tasks/ -H "Authorization: Bearer <access_token>" -H "Content-Type: application/json" -d '{"title": "Yangi vazifa", "description": "Test vazifa"}'
```
📌 Celery worker terminalida yoki `debug.log` faylida quyidagi logni ko'rasiz:
```
[Celery] Yangi vazifa yaratildi: Yangi vazifa (ID: 1, Owner: user1)
```

### ❇️ **Vazifa yangilash va signal sinovi**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
📌 **Usul**: PATCH  
```bash
curl -X PATCH http://127.0.0.1:8000/tasks/1/ -H "Authorization: Bearer <access_token>" -H "Content-Type: application/json" -d '{"completed": true}'
```
📌 Log (Celery emas, signal orqali):
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

## ✅ 12. ADMIN PANELI ORQALI SINOV (IXTIYORIY)
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

## ✅ 13. SINOV UCHUN MASALALAR
📌 Celery vazifalarini sinash uchun quyidagi amallarni bajarib ko'ring:
1. `/tasks/` manzilida yangi vazifa qo'shing va `debug.log` yoki Celery terminalida logni tekshiring.
2. Admin panelida vazifa qo'shing va Celery logini ko'ring.
3. Celery workerni o'chirib, vazifa qo'shing va logning kechiktirilganligini tekshiring.
4. `/tasks/1/` manzilida vazifani yangilang va signal logini tekshiring.
