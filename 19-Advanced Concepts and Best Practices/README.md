# 🌐 DJANGO REST FRAMEWORK ASOSLARI

# 🧩 19-DARS: ILG'OR TUSHUNCHALAR VA ENG YAXSHI AMALIYOTLAR

Bu darsda Django REST Framework (DRF) da ilg'or tushunchalar va eng yaxshi amaliyotlarni bosqichma-bosqich o'rganamiz. Biz API versioning, DRF-spectacular bilan avtomatik API hujjatlari, maxsus xatolik boshqaruvi va loyiha sifatini oshirish uchun eng yaxshi amaliyotlarni ko'rib chiqamiz. Har bir qadam tushunarli va faqat ilg'or tushunchalar va amaliyotlarga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli, JWT autentifikatsiyasi, ruxsatlar, sahifalash, Celery, keshlash, WebSocket va Docker asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan (shu jumladan `owner` maydoni).
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- `TaskViewSet` `myapp/views.py` faylida sozlangan (JWT autentifikatsiyasi, ruxsatlar, filtrlash, sahifalash, keshlash va WebSocket bilan).
- Signallar, Celery, WebSocket va testlar `myapp/signals.py`, `myapp/tasks.py`, `myapp/consumers.py` va `myapp/tests.py` fayllarida sozlangan.
- Docker sozlamalari `Dockerfile` va `docker-compose.yml` fayllarida mavjud.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. ILG'OR TUSHUNCHALAR VA ENG YAXSHI AMALIYOTLAR NI TUSHUNISH
📌 **Ilg'or tushunchalar**:
- **API versioning**: API ning turli versiyalarini qo'llab-quvvatlash.
- **DRF-spectacular**: Avtomatik API hujjatlari (OpenAPI/Swagger) yaratish.
- **Maxsus xatolik boshqaruvi**: API xatolarini markazlashgan tarzda boshqarish.
📌 **Eng yaxshi amaliyotlar**:
- Kod tashkiloti: Loyiha tuzilmasini soddalashtirish va qayta ishlatish.
- Xavfsizlik: API va ma'lumotlarni himoya qilish.
- Logging: Tahlil va xatolarni kuzatish uchun yaxshilangan logging.
- Test qamrovi: To'liq funksionallikni sinash.
📌 Ushbu darsda ushbu tushunchalarni loyihamizga qo'llaymiz.

## ✅ 3. PAKETLARNI O'RNATISH
📌 API versioning va DRF-spectacular uchun qo'shimcha paketlarni o'rnatamiz:
```bash
pip install drf-spectacular
```
📌 `requirements.txt` faylini yangilang:
```
django==4.2
djangorestframework==3.14
django-filter==23.2
djangorestframework-simplejwt==5.2
channels==4.0
channels-redis==4.0
django-redis==5.2
celery==5.2
redis==4.5
psycopg2-binary==2.9
daphne==4.0
drf-spectacular==0.26
```

## ✅ 4. LOYIHA SOZLAMALARINI YANGILASH
📌 `myproject/settings.py` faylida API versioning va DRF-spectacular sozlamalarini qo'shamiz:
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
    'drf_spectacular',
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
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.URLPathVersioning',  # API versioning
    'DEFAULT_VERSION': 'v1',  # Standart versiya
    'ALLOWED_VERSIONS': ['v1', 'v2'],  # Ruxsat etilgan versiyalar
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',  # DRF-spectacular
}

# PostgreSQL sozlamalari
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydb',
        'USER': 'myuser',
        'PASSWORD': 'mypassword',
        'HOST': 'db',
        'PORT': '5432',
    }
}

# Celery sozlamalari
CELERY_BROKER_URL = 'redis://redis:6379/0'
CELERY_RESULT_BACKEND = 'redis://redis:6379/0'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_TIMEZONE = 'Asia/Tashkent'
CELERY_TASK_ALWAYS_EAGER = False

# Keshlash sozlamalari
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://redis:6379/1',
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
            'hosts': [('redis', 6379)],
        },
    },
}

# Logging sozlamalari
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': 'debug.log',
            'formatter': 'verbose',
        },
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
    },
    'loggers': {
        'myapp': {
            'handlers': ['file', 'console'],
            'level': 'INFO',
            'propagate': True,
        },
        'django': {
            'handlers': ['file', 'console'],
            'level': 'INFO',
            'propagate': True,
        },
    },
}
```
📌 **Tushuntirish**:
- `DEFAULT_VERSIONING_CLASS`: URL da versiyalashni yoqadi (masalan, `/v1/tasks/`).
- `DEFAULT_SCHEMA_CLASS`: DRF-spectacular ni API hujjatlari uchun sozlaydi.
- `LOGGING`: Formatlangan loglar uchun `verbose` formater qo'shildi.

## ✅ 5. API VERSIONING NI SOZLASH
📌 `myproject/urls.py` faylini yangilaymiz va versiyalashni qo'llab-quvvatlaymiz:
```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView
from myapp.views import TaskViewSet
from drf_spectacular.views import SpectacularAPIView, SpectacularSwaggerView

router = DefaultRouter()
router.register(r'tasks', TaskViewSet)

urlpatterns = [
    path('', include('myapp.urls')),  # WebSocket sahifasi uchun
    path('v1/', include(router.urls)),  # v1 versiyasi
    path('v2/', include(router.urls)),  # v2 versiyasi (keyinchalik farqlarni qo'shish mumkin)
    path('v1/api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('v1/api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    path('v1/schema/', SpectacularAPIView.as_view(), name='schema'),  # API sxemasi
    path('v1/docs/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),  # Swagger UI
]
```
📌 **Tushuntirish**:
- `v1/` va `v2/`: API versiyalari uchun prefikslar.
- `SpectacularAPIView`: OpenAPI sxemasini yaratadi.
- `SpectacularSwaggerView`: Swagger UI ni ta'minlaydi.

## ✅ 6. MAXSUS XATOLIK BOSHQARUVI
📌 `myapp/exceptions.py` faylini yarating va maxsus xatolik sinfini qo'shing:
```python
from rest_framework.views import exception_handler
from rest_framework.response import Response
from rest_framework import status

def custom_exception_handler(exc, context):
    response = exception_handler(exc, context)
    
    if response is not None:
        custom_response = {
            'error': {
                'status_code': response.status_code,
                'message': str(exc),
                'detail': response.data
            }
        }
        response.data = custom_response
    
    return response
```
📌 `myproject/settings.py` faylida xatolik boshqaruvchisini sozlang:
```python
REST_FRAMEWORK = {
    ...
    'EXCEPTION_HANDLER': 'myapp.exceptions.custom_exception_handler',
}
```
📌 **Tushuntirish**:
- `custom_exception_handler`: API xatolarini moslashtirilgan formatda qaytaradi (masalan, `{'error': {...}}`).

## ✅ 7. KOD TASHKILOTI VA ENG YAXSHI AMALIYOTLAR
📌 Quyidagi amaliyotlarni qo'llaymiz:
- **Modulli tuzilma**: Kodni `models`, `serializers`, `views`, `permissions`, `pagination`, `signals`, `tasks`, `consumers` va `exceptions` kabi alohida fayllarga ajratdik.
- **Xavfsizlik**:
  - JWT autentifikatsiyasi va `IsOwnerOrReadOnly` ruxsatlarini qo'lladik.
  - `ALLOWED_HOSTS` ni sozlash:
    ```python
    ALLOWED_HOSTS = ['localhost', '127.0.0.1', 'your-domain.com']
    ```
  - HTTPS ni Nginx orqali sozlash (ishlab chiqarishda).
- **Logging**: Formatlangan loglar qo'shildi (`verbose` formater).
- **Test qamrovi**: Oldingi darsda yozilgan testlar loyiha funksionalligini qamrab oladi.
- **Ma'lumotlar bazasi optimallashtirish**: `select_related` qo'llanildi.

## ✅ 8. DRF-SPECTACULAR BILAN API HUJJATLARI
📌 `myapp/views.py` faylida `TaskViewSet` ni DRF-spectacular uchun hujjatlaymiz:
```python
from rest_framework import viewsets
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.filters import SearchFilter, OrderingFilter
from django.views.decorators.cache import cache_page
from django.utils.decorators import method_decorator
from .models import Task
from .serializers import TaskSerializer
from .permissions import IsOwnerOrReadOnly
from .pagination import CustomTaskPagination
from rest_framework.permissions import IsAuthenticated
from drf_spectacular.utils import extend_schema, OpenApiParameter

class TaskViewSet(viewsets.ModelViewSet):
    queryset = Task.objects.all().select_related('owner')
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

    @method_decorator(cache_page(60 * 5))
    @extend_schema(
        description='Barcha vazifalarni ro\'yxatini qaytaradi, sahifalash va filtrlash qo\'llaniladi.',
        parameters=[
            OpenApiParameter(name='completed', description='Vazifa holati (true/false)', required=False, type=bool),
            OpenApiParameter(name='search', description='Sarlavha yoki tavsif bo\'yicha qidirish', required=False, type=str),
        ]
    )
    def list(self, request, *args, **kwargs):
        return super().list(request, *args, **kwargs)
```
📌 **Tushuntirish**:
- `@extend_schema`: API endpointi uchun hujjatlashtirish qo'shadi.
- `OpenApiParameter`: Filtr va qidiruv parametrlari uchun hujjatlashtirish.

## ✅ 9. TESTLARNI YANGILASH
📌 `myapp/tests.py` fayliga API versioning va xatolik boshqaruvini sinash uchun testlar qo'shamiz:
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
import json
import logging

logger = logging.getLogger(__name__)

# Oldingi testlar (TaskModelTest, TaskSerializerTest, TaskSignalTest, TaskAPITest, TaskWebSocketTest) shu yerda qoladi

class TaskAdvancedTest(APITestCase):
    def setUp(self):
        self.user = User.objects.create_user(username='testuser', password='testpass123')
        self.token = RefreshToken.for_user(self.user).access_token
        self.client.credentials(HTTP_AUTHORIZATION=f'Bearer {self.token}')
        cache.clear()

    def test_api_versioning(self):
        url_v1 = reverse('task-list', kwargs={'version': 'v1'})
        response_v1 = self.client.get(url_v1)
        self.assertEqual(response_v1.status_code, status.HTTP_200_OK)

        url_v2 = reverse('task-list', kwargs={'version': 'v2'})
        response_v2 = self.client.get(url_v2)
        self.assertEqual(response_v2.status_code, status.HTTP_200_OK)

    def test_custom_exception_handler(self):
        url = reverse('task-detail', kwargs={'version': 'v1', 'pk': 999})  # Mavjud bo'lmagan ID
        response = self.client.get(url)
        self.assertEqual(response.status_code, status.HTTP_404_NOT_FOUND)
        self.assertIn('error', response.data)
        self.assertEqual(response.data['error']['status_code'], 404)
        self.assertIn('detail', response.data['error'])

    def test_api_documentation(self):
        url = reverse('schema', kwargs={'version': 'v1'})
        response = self.client.get(url)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
```
📌 **Tushuntirish**:
- `test_api_versioning`: `v1` va `v2` versiyalarini sinaydi.
- `test_custom_exception_handler`: Maxsus xatolik formatini tekshiradi.
- `test_api_documentation`: API sxemasining mavjudligini sinaydi.

## ✅ 10. DOCKER SOZLAMALARINI TEKSHIRISH
📌 `docker-compose.yml` faylida o'zgarishlar talab qilinmaydi, lekin `requirements.txt` yangilanganligi uchun konteynerni qayta quring:
```bash
docker-compose up --build
```

## ✅ 11. SINOV UCHUN MASALALAR
📌 Ilg'or tushunchalar va amaliyotlarni sinash uchun quyidagi amallarni bajarib ko'ring:
1. `http://localhost/v1/docs/` manzilida Swagger UI ni oching va API hujjatlarini ko'ring.
2. `http://localhost/v1/tasks/` va `http://localhost/v2/tasks/` manzillarini sinab, versiyalashni tekshiring.
3. Mavjud bo'lmagan vazifani so'rang (`/v1/tasks/999/`) va maxsus xatolik formatini ko'ring.
4. `python manage.py test` bilan yangi testlarni sinab ko'ring.
5. Loglarni `debug.log` faylida tekshiring va formatlangan xabarlarni ko'ring.

