# 🌐 DJANGO REST FRAMEWORK ASOSLARI

# 🧩 17-DARS: WEBSOCKETS VA DJANGO CHANNELS BILAN ISHLASH

Bu darsda Django REST Framework (DRF) va Django Channels yordamida WebSocket integratsiyasini sozlashni bosqichma-bosqich o'rganamiz. WebSocket real vaqtda ikki tomonlama aloqani ta'minlaydi, masalan, vazifa yaratilganda foydalanuvchilarga bildirishnoma yuborish uchun. Har bir qadam tushunarli va faqat WebSocket va Django Channels ga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli, JWT autentifikatsiyasi, ruxsatlar, sahifalash, Celery va keshlash asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan (shu jumladan `owner` maydoni).
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- `TaskViewSet` `myapp/views.py` faylida sozlangan (JWT autentifikatsiyasi, ruxsatlar, filtrlash, sahifalash, Celery va keshlash bilan).
- Signallar va Celery `myapp/signals.py` va `myapp/tasks.py` fayllarida sozlangan.
- `/tasks/` endpointi `myproject/urls.py` faylida sozlangan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. WEBSOCKET VA DJANGO CHANNELS NI TUSHUNISH
📌 **WebSocket**: HTTP dan farqli o'laroq, server va klient o'rtasida doimiy, ikki tomonlama aloqani ta'minlaydi. Bu real vaqtda ma'lumot uzatish (masalan, bildirishnomalar) uchun ideal.
📌 **Django Channels**: Django da WebSocket va asinxron vazifalarni boshqarish uchun ishlatiladi. U Redis kabi xabar brokeri orqali ishlaydi.
📌 Ushbu darsda vazifa yaratilganda barcha ulangan foydalanuvchilarga WebSocket orqali bildirishnoma yuboramiz.

## ✅ 3. DJANGO CHANNELS VA REDIS NI O'RNATISH
📌 Quyidagi paketlarni o'rnatamiz:
```bash
pip install channels channels-redis
```
📌 Redis allaqachon Celery va keshlash uchun o'rnatilgan. Redis serveri ishlayotganligiga ishonch hosil qiling:
```bash
redis-server
redis-cli ping  # Javob: PONG
```

## ✅ 4. LOYIHA SOZLAMALARINI YANGILASH
📌 `myproject/settings.py` faylida Channels va Redis sozlamalarini qo'shamiz:
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'rest_framework.authtoken',
    'django_filters',
    'channels',
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
CELERY_BROKER_URL = 'redis://localhost:6379/0'
CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_TIMEZONE = 'Asia/Tashkent'
CELERY_TASK_ALWAYS_EAGER = True

# Keshlash sozlamalari
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}

# Channels sozlamalari
ASGI_APPLICATION = 'myproject.asgi.application'
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            'hosts': [('127.0.0.1', 6379)],
        },
    },
}
```
📌 **Tushuntirish**:
- `channels` ni `INSTALLED_APPS` ga qo'shdik.
- `ASGI_APPLICATION`: Django Channels uchun ASGI ilovasini belgilaydi.
- `CHANNEL_LAYERS`: Redis ni WebSocket xabarlari uchun ishlatadi.

## ✅ 5. ASGI SOZLASH
📌 `myproject/asgi.py` faylini yangilaymiz yoki yarating:
```python
import os
from django.core.asgi import get_asgi_application
from channels.routing import ProtocolTypeRouter, URLRouter
from channels.auth import AuthMiddlewareStack
import myapp.routing

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

application = ProtocolTypeRouter({
    'http': get_asgi_application(),
    'websocket': AuthMiddlewareStack(
        URLRouter(
            myapp.routing.websocket_urlpatterns
        )
    ),
})
```
📌 **Tushuntirish**:
- `ProtocolTypeRouter`: HTTP va WebSocket so'rovlarini boshqaradi.
- `AuthMiddlewareStack`: WebSocket ulanishlarida autentifikatsiyani qo'llab-quvvatlaydi.

## ✅ 6. WEBSOCKET ROUTING
📌 `myapp/routing.py` faylini yarating va WebSocket marshrutlarini sozlang:
```python
from django.urls import re_path
from . import consumers

websocket_urlpatterns = [
    re_path(r'ws/tasks/$', consumers.TaskConsumer.as_asgi()),
]
```
📌 **Tushuntirish**:
- `ws/tasks/`: WebSocket ulanishi uchun endpoint.
- `TaskConsumer`: WebSocket xabarlarini boshqaradi.

## ✅ 7. WEBSOCKET CONSUMER YARATISH
📌 `myapp/consumers.py` faylini yarating va quyidagi kodni qo'shing:
```python
from channels.generic.websocket import AsyncWebsocketConsumer
import json

class TaskConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        await self.channel_layer.group_add('tasks', self.channel_name)
        await self.accept()

    async def disconnect(self, close_code):
        await self.channel_layer.group_discard('tasks', self.channel_name)

    async def task_notification(self, event):
        await self.send(text_data=json.dumps({
            'message': event['message']
        }))
```
📌 **Tushuntirish**:
- `connect`: Foydalanuvchi WebSocket ga ulanganda `tasks` guruhiga qo'shiladi.
- `disconnect`: Ulanish uzilganda guruhdan chiqariladi.
- `task_notification`: Guruhdagi barcha foydalanuvchilarga xabar yuboradi.

## ✅ 8. SIGNAL ORQALI WEBSOCKET BILDIRISHNOMASI
📌 `myapp/signals.py` faylini yangilaymiz va vazifa yaratilganda WebSocket orqali bildirishnoma yuboramiz:
```python
from django.db.models.signals import post_save, post_delete
from django.dispatch import receiver
from .models import Task
from .tasks import log_task_creation
from channels.layers import get_channel_layer
from asgiref.sync import async_to_sync
import logging

logger = logging.getLogger(__name__)

@receiver(post_save, sender=Task)
def handle_task_creation(sender, instance, created, **kwargs):
    if created:
        # Celery vazifasini fon rejimida ishga tushirish
        log_task_creation.delay(instance.id, instance.title, instance.owner.username)
        # WebSocket orqali bildirishnoma yuborish
        channel_layer = get_channel_layer()
        async_to_sync(channel_layer.group_send)(
            'tasks',
            {
                'type': 'task_notification',
                'message': f"Yangi vazifa yaratildi: {instance.title} (Owner: {instance.owner.username})"
            }
        )
    else:
        logger.info(f"Vazifa yangilandi: {instance.title} (ID: {instance.id}, Owner: {instance.owner.username})")

@receiver(post_delete, sender=Task)
def log_task_deletion(sender, instance, **kwargs):
    logger.info(f"Vazifa o'chirildi: {instance.title} (ID: {instance.id}, Owner: {instance.owner.username})")
```
📌 **Tushuntirish**:
- `group_send`: `tasks` guruhidagi barcha ulangan foydalanuvchilarga xabar yuboradi.
- `async_to_sync`: Asinxron Channels funksiyasini sinxron signalda ishlatish uchun.

## ✅ 9. KLIENT-SAYT WEBSOCKET INTEGRATSIYASI
📌 WebSocket ni sinash uchun oddiy HTML sahifasi yarating. `myapp/templates/index.html` faylini yarating:
```html
<!DOCTYPE html>
<html>
<head>
    <title>WebSocket Test</title>
</head>
<body>
    <h1>WebSocket Bildirishnomalari</h1>
    <div id="notifications"></div>

    <script>
        const socket = new WebSocket('ws://localhost:8000/ws/tasks/');
        
        socket.onmessage = function(event) {
            const data = JSON.parse(event.data);
            const notifications = document.getElementById('notifications');
            const p = document.createElement('p');
            p.textContent = data.message;
            notifications.appendChild(p);
        };

        socket.onopen = function(event) {
            console.log('WebSocket ulandi');
        };

        socket.onclose = function(event) {
            console.log('WebSocket uzildi');
        };
    </script>
</body>
</html>
```

📌 `myapp/urls.py` faylini yarating va HTML sahifasi uchun URL qo'shing:
```python
from django.urls import path
from django.views.generic import TemplateView

urlpatterns = [
    path('', TemplateView.as_view(template_name='index.html'), name='index'),
]
```

📌 `myproject/urls.py` faylini yangilang:
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
    path('ws/', include('myapp.urls')),  # WebSocket sahifasi uchun
]
```

## ✅ 10. WEBSOCKET NI SINAB KO'RISH
📌 Serverni ishga tushiring (Django Channels Daphne bilan ishlaydi):
```bash
pip install daphne
daphne -b 0.0.0.0 -p 8000 myproject.asgi:application
```
📌 Redis serveri ishlayotganligiga ishonch hosil qiling:
```bash
redis-server
```
📌 Celery worker ni ishga tushiring (ixtiyoriy):
```bash
celery -A myproject worker --loglevel=info
```

### ❇️ **Foydalanuvchi va token yaratish**:
📌 Agar hali yaratilmagan bo'lsa, foydalanuvchi yarating:
```bash
python manage.py createsuperuser  # username: testuser, password: testpass123
```
📌 JWT token oling:
```bash
curl -X POST http://127.0.0.1:8000/api/token/ -H "Content-Type: application/json" -d '{"username": "testuser", "password": "testpass123"}'
```

### ❇️ **WebSocket sinovi**:
1. Brauzerda `http://localhost:8000/ws/` manzilini oching.
2. Yangi vazifa qo'shing:
   ```bash
   curl -X POST http://127.0.0.1:8000/tasks/ -H "Authorization: Bearer <access_token>" -H "Content-Type: application/json" -d '{"title": "Yangi vazifa", "description": "Test"}'
   ```
3. Brauzerda bildirishnoma paydo bo'ladi: `Yangi vazifa yaratildi: Yangi vazifa (Owner: testuser)`.

## ✅ 11. TESTLARNI YANGILASH
📌 WebSocket ni sinash uchun `myapp/tests.py` fayliga test qo'shamiz:
```python
from django.test import TestCase
from rest_framework.test import APITestCase
from rest_framework import status
from django.urls import reverse
from django.contrib.auth.models import User
from rest_framework_simplejwt.tokens import RefreshToken
from .models import Task
from .serializers import TaskSerializer
from django.core.cache import cache
from channels.testing import WebsocketCommunicator
from myapp.consumers import TaskConsumer
import logging

logger = logging.getLogger(__name__)

# Oldingi testlar (TaskModelTest, TaskSerializerTest, TaskSignalTest, TaskAPITest) shu yerda qoladi

class TaskWebSocketTest(TestCase):
    async def test_task_notification(self):
        communicator = WebsocketCommunicator(TaskConsumer.as_asgi(), "/ws/tasks/")
        connected, subprotocol = await communicator.connect()
        self.assertTrue(connected)

        # Foydalanuvchi va vazifa yaratish
        user = User.objects.create_user(username='testuser', password='testpass123')
        task = Task.objects.create(title='Test vazifa', owner=user)

        # Bildirishnoma olish
        response = await communicator.receive_from()
        data = json.loads(response)
        self.assertEqual(data['message'], f"Yangi vazifa yaratildi: Test vazifa (Owner: testuser)")

        await communicator.disconnect()
```
📌 Testlarni ishga tushirish:
```bash
python manage.py test
```

## ✅ 12. SINOV UCHUN MASALALAR
📌 WebSocket ni sinash uchun quyidagi amallarni bajarib ko'ring:
1. `http://localhost:8000/ws/` manzilida WebSocket sahifasini oching.
2. `/tasks/` manzilida yangi vazifa qo'shing va brauzerda bildirishnoma oling.
3. Bir nechta brauzer oynasini ochib, guruh bildirishnomasini sinab ko'ring.
4. `python manage.py test` bilan WebSocket testini sinab ko'ring.

