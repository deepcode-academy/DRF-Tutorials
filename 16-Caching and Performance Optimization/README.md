# 🧩 16-DARS: KESHLASH VA ISHLASHNI OPTIMALASHTIRISH

Bu darsda Django REST Framework (DRF) yordamida keshlash (caching) va ishlashni optimallashtirish usullarini bosqichma-bosqich o'rganamiz. Keshlash API javoblarini tezkorlashtiradi va server yukini kamaytiradi. Har bir qadam tushunarli va faqat keshlash va ishlash optimallashtirishga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli, JWT autentifikatsiyasi, ruxsatlar, sahifalash, signallar va Celery asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan (shu jumladan `owner` maydoni).
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- `TaskViewSet` `myapp/views.py` faylida sozlangan (JWT autentifikatsiyasi, ruxsatlar, filtrlash, sahifalash va Celery bilan).
- Signallar va Celery `myapp/signals.py` va `myapp/tasks.py` fayllarida sozlangan.
- Testlar `myapp/tests.py` faylida yozilgan.
- `/tasks/` endpointi `myproject/urls.py` faylida sozlangan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. KESHLASH VA ISHLASH OPTIMALASHTIRISH NI TUSHUNISH
📌 **Keshlash**: Tez-tez so'raladigan ma'lumotlarni vaqtincha saqlab, ma'lumotlar bazasiga murojaatlarni kamaytiradi, bu esa javob vaqtini qisqartiradi.
📌 **Ishlash optimallashtirish**: API so'rovlarini tezkorlashtirish uchun turli usullar (masalan, keshlash, ma'lumotlar bazasi so'rovlarini optimallashtirish) qo'llaniladi.
📌 DRF keshlash turlari:
- **Per-view keshlash**: Muayyan view uchun keshlash.
- **Per-site keshlash**: Butun sayt uchun global keshlash.
- **Backend keshlash**: Redis yoki Memcached kabi tizimlar bilan.
📌 Ushbu darsda Redis bilan keshlash va ma'lumotlar bazasi so'rovlarini optimallashtirishga e'tibor qaratamiz.

## ✅ 3. REDIS KESHLASH UCHUN SOZLASH
📌 Oldingi darsda Redis allaqachon Celery uchun o'rnatilgan. Agar hali o'rnatilmagan bo'lsa:
```bash
pip install redis
sudo apt-get install redis-server  # Ubuntu uchun
```
📌 Redis serverini ishga tushiring:
```bash
redis-server
redis-cli ping  # Javob: PONG
```

## ✅ 4. LOYIHA SOZLAMALARINI YANGILASH
📌 `myproject/settings.py` faylida keshlash sozlamalarini qo'shamiz:
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
        'LOCATION': 'redis://127.0.0.1:6379/1',  # Celery dan farqli database (1)
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}
```
📌 **Tushuntirish**:
- `CACHES`: Redis ni kesh backend sifatida sozlaydi. Celery dan farqli database (1) ishlatiladi.
- `LOCATION`: Redis server manzili, database 1 uchun.

## ✅ 5. KESHLASHNI VIEWSET GA QO'SHISH
📌 `TaskViewSet` da `/tasks/` endpointi uchun keshlashni qo'shamiz. `myapp/views.py` faylini yangilaymiz:
```python
from rest_framework import viewsets
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.filters import SearchFilter, OrderingFilter
from rest_framework.decorators import action
from django.views.decorators.cache import cache_page
from django.utils.decorators import method_decorator
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

    @method_decorator(cache_page(60 * 5))  # 5 daqiqa keshlash
    def list(self, request, *args, **kwargs):
        return super().list(request, *args, **kwargs)
```
📌 **Tushuntirish**:
- `cache_page(60 * 5)`: `/tasks/` endpointi uchun javobni 5 daqiqa keshlaydi.
- `@method_decorator`: DRF viewset metodlarida keshlashni qo'llash uchun ishlatiladi.
- Faqat `list` (GET /tasks/) metodi keshlanadi, chunki u eng ko'p so'raladigan operatsiya.

## ✅ 6. MA'LUMOTLAR BAZASI SO'ROVLARINI OPTIMALASHTIRISH
📌 Ma'lumotlar bazasi so'rovlarini kamaytirish uchun `Task` modelidagi `owner` maydoniga `select_related` qo'shamiz:
```python
class TaskViewSet(viewsets.ModelViewSet):
    queryset = Task.objects.all().select_related('owner')  # owner maydonini oldindan yuklash
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
    def list(self, request, *args, **kwargs):
        return super().list(request, *args, **kwargs)
```
📌 **Tushuntirish**:
- `select_related('owner')`: `owner` maydonini oldindan yuklaydi, bu esa qo'shimcha ma'lumotlar bazasi so'rovlarini kamaytiradi.

## ✅ 7. TESTLARNI YANGILASH
📌 Keshlashni sinash uchun `myapp/tests.py` fayliga yangi test qo'shamiz:
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
import logging

logger = logging.getLogger(__name__)

# Oldingi testlar (TaskModelTest, TaskSerializerTest, TaskSignalTest) shu yerda qoladi

class TaskAPITest(APITestCase):
    def setUp(self):
        self.user = User.objects.create_user(username='testuser', password='testpass123')
        self.other_user = User.objects.create_user(username='otheruser', password='otherpass123')
        self.task = Task.objects.create(
            title='Test vazifa',
            description='Test tavsifi',
            owner=self.user
        )
        self.token = RefreshToken.for_user(self.user).access_token
        self.client.credentials(HTTP_AUTHORIZATION=f'Bearer {self.token}')
        cache.clear()  # Testdan oldin keshni tozalash

    def test_get_tasks_list(self):
        url = reverse('task-list')
        response = self.client.get(url)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 1)
        self.assertEqual(response.data['results'][0]['title'], 'Test vazifa')

    def test_create_task(self):
        url = reverse('task-list')
        data = {'title': 'Yangi vazifa', 'description': 'Yangi tavsif'}
        response = self.client.post(url, data, format='json')
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(Task.objects.count(), 2)
        self.assertEqual(Task.objects.latest('id').title, 'Yangi vazifa')

    def test_update_task_owner_only(self):
        url = reverse('task-detail', kwargs={'pk': self.task.id})
        data = {'completed': True}
        response = self.client.patch(url, data, format='json')
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.task.refresh_from_db()
        self.assertTrue(self.task.completed)

        other_token = RefreshToken.for_user(self.other_user).access_token
        self.client.credentials(HTTP_AUTHORIZATION=f'Bearer {other_token}')
        response = self.client.patch(url, data, format='json')
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)

    def test_delete_task_owner_only(self):
        url = reverse('task-detail', kwargs={'pk': self.task.id})
        response = self.client.delete(url)
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
        self.assertEqual(Task.objects.count(), 0)

        self.task = Task.objects.create(title='Yana vazifa', owner=self.user)
        other_token = RefreshToken.for_user(self.other_user).access_token
        self.client.credentials(HTTP_AUTHORIZATION=f'Bearer {other_token}')
        response = self.client.delete(url)
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)

    def test_pagination(self):
        for i in range(5):
            Task.objects.create(title=f'Vazifa {i}', owner=self.user)
        url = reverse('task-list')
        response = self.client.get(url)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 3)
        self.assertIn('next', response.data)
        self.assertIsNone(response.data['previous'])

    def test_filtering(self):
        Task.objects.create(title='Bajarilgan vazifa', completed=True, owner=self.user)
        url = reverse('task-list') + '?completed=true'
        response = self.client.get(url)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 1)
        self.assertEqual(response.data['results'][0]['title'], 'Bajarilgan vazifa')

    def test_caching(self):
        url = reverse('task-list')
        # Birinchi so'rov
        response1 = self.client.get(url)
        self.assertEqual(response1.status_code, status.HTTP_200_OK)
        # Keshdan ikkinchi so'rov
        response2 = self.client.get(url)
        self.assertEqual(response2.status_code, status.HTTP_200_OK)
        self.assertEqual(response1.data, response2.data)  # Keshlangan javob bir xil
        # Yangi vazifa qo'shish keshni yangilamaydi
        Task.objects.create(title='Kesh sinovi', owner=self.user)
        response3 = self.client.get(url)
        self.assertEqual(len(response3.data['results']), len(response1.data['results']))  # Keshlangan natija
```
📌 **Tushuntirish**:
- `test_caching`: `/tasks/` endpointining keshlanganligini sinaydi.
- `cache.clear()`: Testdan oldin keshni tozalaydi.
- Keshlangan javob bir xil bo'lishi va yangi vazifa qo'shilganda keshning yangilanmasligi tekshiriladi.

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

## ✅ 9. KESHLASH VA ISHLASHNI SINAB KO'RISH
📌 Serverni ishga tushiring:
```bash
python manage.py runserver
```
📌 Redis serverini ishga tushiring:
```bash
redis-server
```
📌 Celery worker ni ishga tushiring (ixtiyoriy, testlar uchun shart emas):
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

### ❇️ **Keshlash sinovi**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
📌 **Usul**: GET  
```bash
curl http://127.0.0.1:8000/tasks/ -H "Authorization: Bearer <access_token>"
```
📌 Birinchi so'rov ma'lumotlar bazasidan olinadi, keyingi so'rovlar 5 daqiqa davomida keshdan qaytariladi. Yangi vazifa qo'shing:
```bash
curl -X POST http://127.0.0.1:8000/tasks/ -H "Authorization: Bearer <access_token>" -H "Content-Type: application/json" -d '{"title": "Yangi vazifa", "description": "Test"}'
```
📌 `/tasks/` ni qayta so'rang — kesh tufayli yangi vazifa ko'rinmaydi.

### ❇️ **Testlarni ishga tushirish**:
```bash
python manage.py test
```
📌 `test_caching` testi keshning to'g'ri ishlashini tasdiqlaydi.

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

📌 `http://127.0.0.1:8000/admin/` manzilida vazifalarni qo'shing yoki tahrirlang.

## ✅ 11. SINOV UCHUN MASALALAR
📌 Keshlash va ishlashni sinash uchun quyidagi amallarni bajarib ko'ring:
1. `/tasks/` manziliga bir necha marta GET so'rov yuboring va javob vaqtini solishtiring.
2. Yangi vazifa qo'shing va `/tasks/` manzilida kesh tufayli yangi vazifa ko'rinmasligini tekshiring.
3. `python manage.py test` bilan `test_caching` testini sinab ko'ring.
4. `select_related` effekti uchun ma'lumotlar bazasi so'rovlarini (Django Debug Toolbar bilan) tekshiring.
