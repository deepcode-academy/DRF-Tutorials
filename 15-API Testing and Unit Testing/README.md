# 🌐 DJANGO REST FRAMEWORK ASOSLARI

# 🧩 15-DARS: API TESTLARI VA UNIT TESTLARI BILAN ISHLASH

Bu darsda Django REST Framework (DRF) yordamida API testlari va unit testlarini yozishni bosqichma-bosqich o'rganamiz. Testlar loyihaning to'g'ri ishlashini ta'minlaydi va kod sifatini oshiradi. Har bir qadam tushunarli va faqat API va unit testlariga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli, JWT autentifikatsiyasi, ruxsatlar, sahifalash va Celery asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan (shu jumladan `owner` maydoni).
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- `TaskViewSet` `myapp/views.py` faylida sozlangan (JWT autentifikatsiyasi, ruxsatlar, filtrlash, sahifalash va Celery bilan).
- Signallar va Celery `myapp/signals.py` va `myapp/tasks.py` fayllarida sozlangan.
- `/tasks/` endpointi `myproject/urls.py` faylida sozlangan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. API VA UNIT TESTLARINI TUSHUNISH
📌 **Unit testlar**: Kodning alohida qismlarini (masalan, model yoki serializer metodlari) sinash uchun ishlatiladi.
📌 **API testlari**: API endpointlarining to'g'ri ishlashini, HTTP javoblarini va ma'lumotlar formatini sinash uchun ishlatiladi.
📌 Django `TestCase` va DRFning `APITestCase` sinflaridan foydalanamiz:
- `TestCase`: Model va umumiy logikani sinash uchun.
- `APITestCase`: API so'rovlarini (GET, POST, PATCH, DELETE) sinash uchun.
📌 Testlar loyihaning funksionalligini (masalan, JWT autentifikatsiyasi, ruxsatlar, sahifalash) va Celery signal integratsiyasini tekshiradi.

## ✅ 3. TESTLAR UCHUN SOZLAMALAR
📌 Testlar yozish uchun qo'shimcha paketlar o'rnatish shart emas, chunki Django va DRF o'z ichida test vositalarini ta'minlaydi.
📌 Redis va Celery test paytida ishlayotgan bo'lishi kerak. Testlarda Celery vazifalarini sinxron tarzda bajarish uchun `CELERY_TASK_ALWAYS_EAGER` sozlamasini qo'shamiz.

📌 `myproject/settings.py` faylida test sozlamasini qo'shing:
```python
# ... Boshqa sozlamalar ...
CELERY_TASK_ALWAYS_EAGER = True  # Testlarda Celery vazifalari sinxron bajariladi
```

## ✅ 4. TEST FAYLINI YARATISH
📌 `myapp/tests.py` faylini yarating va quyidagi testlarni qo'shing. Testlar model, serializer, API endpointlari va Celery signal integratsiyasini sinaydi:
```python
from django.test import TestCase
from rest_framework.test import APITestCase
from rest_framework import status
from django.urls import reverse
from django.contrib.auth.models import User
from rest_framework_simplejwt.tokens import RefreshToken
from .models import Task
from .serializers import TaskSerializer
import logging

# Logger sozlash
logger = logging.getLogger(__name__)

class TaskModelTest(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(username='testuser', password='testpass123')
        self.task = Task.objects.create(
            title='Test vazifa',
            description='Test tavsifi',
            completed=False,
            owner=self.user
        )

    def test_task_creation(self):
        self.assertEqual(self.task.title, 'Test vazifa')
        self.assertEqual(self.task.owner.username, 'testuser')
        self.assertFalse(self.task.completed)
        self.assertEqual(str(self.task), 'Test vazifa')

class TaskSerializerTest(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(username='testuser', password='testpass123')
        self.task = Task.objects.create(
            title='Test vazifa',
            description='Test tavsifi',
            owner=self.user
        )
        self.serializer = TaskSerializer(instance=self.task)

    def test_serializer_contains_expected_fields(self):
        data = self.serializer.data
        self.assertEqual(set(data.keys()), {'id', 'title', 'description', 'completed', 'created_at', 'status', 'owner'})
        self.assertEqual(data['title'], 'Test vazifa')
        self.assertEqual(data['status'], 'Bajarilmagan')
        self.assertEqual(data['owner'], 'testuser')

    def test_serializer_title_validation(self):
        invalid_data = {'title': 'ab', 'description': 'Test'}  # Title 3 belgidan kam
        serializer = TaskSerializer(data=invalid_data)
        self.assertFalse(serializer.is_valid())
        self.assertIn('title', serializer.errors)

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

        # Boshqa foydalanuvchi bilan sinash
        other_token = RefreshToken.for_user(self.other_user).access_token
        self.client.credentials(HTTP_AUTHORIZATION=f'Bearer {other_token}')
        response = self.client.patch(url, data, format='json')
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)

    def test_delete_task_owner_only(self):
        url = reverse('task-detail', kwargs={'pk': self.task.id})
        response = self.client.delete(url)
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
        self.assertEqual(Task.objects.count(), 0)

        # Boshqa foydalanuvchi bilan sinash
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
        self.assertEqual(len(response.data['results']), 3)  # CustomTaskPagination: 3 ta vazifa/sahifa
        self.assertIn('next', response.data)
        self.assertIsNone(response.data['previous'])

    def test_filtering(self):
        Task.objects.create(title='Bajarilgan vazifa', completed=True, owner=self.user)
        url = reverse('task-list') + '?completed=true'
        response = self.client.get(url)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 1)
        self.assertEqual(response.data['results'][0]['title'], 'Bajarilgan vazifa')

class TaskSignalTest(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(username='testuser', password='testpass123')
        # Loggingni sinash uchun
        logger.handlers = []  # Mavjud handlerlarni tozalash
        self.log_output = logging.StreamHandler()
        logger.addHandler(self.log_output)
        logger.setLevel(logging.INFO)

    def test_task_creation_signal(self):
        with self.assertLogs('myapp', level='INFO') as cm:
            Task.objects.create(title='Test vazifa', owner=self.user)
            self.assertIn('Yangi vazifa yaratildi', cm.output[0])

    def test_task_deletion_signal(self):
        task = Task.objects.create(title='Test vazifa', owner=self.user)
        with self.assertLogs('myapp', level='INFO') as cm:
            task.delete()
            self.assertIn('Vazifa o\'chirildi', cm.output[0])
```
📌 **Tushuntirish**:
- `TaskModelTest`: Modelning to'g'ri yaratilishi va `__str__` metodini sinaydi.
- `TaskSerializerTest`: Serializer maydonlari va validatsiyasini sinaydi.
- `TaskAPITest`: API endpointlarini (GET, POST, PATCH, DELETE), JWT autentifikatsiyasini, ruxsatlarni va sahifalashni sinaydi.
- `TaskSignalTest`: Signallar va Celery vazifalarining log yozishini sinaydi.
- `setUp`: Har bir test sinfi uchun boshlang'ich ma'lumotlarni sozlaydi.
- `assertLogs`: Signallarning loglarini tekshiradi.

## ✅ 5. TESTLARNI ISHGA TUSHIRISH
📌 Testlarni ishga tushirish uchun quyidagi buyruqni loyiha papkasida bajaring:
```bash
python manage.py test
```
📌 **Eslatma**: Redis serveri va Celery worker ishlayotgan bo'lishi shart emas, chunki `CELERY_TASK_ALWAYS_EAGER = True` sozlamasi Celery vazifalarini sinxron bajaradi.

## ✅ 6. LOYIHA SOZLAMALARINI TEKSHIRISH
📌 `myproject/settings.py` faylida quyidagi sozlamalar mavjudligiga ishonch hosil qiling:
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

CELERY_BROKER_URL = 'redis://localhost:6379/0'
CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_TIMEZONE = 'Asia/Tashkent'
CELERY_TASK_ALWAYS_EAGER = True  # Testlarda sinxron bajarish
```

## ✅ 7. URL SOZLASH
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

## ✅ 8. TESTLARNI SINAB KO'RISH
📌 Testlarni ishga tushirish uchun:
```bash
python manage.py test
```
📌 Natija: Barcha testlar muvaffaqiyatli o'tsa, "OK" xabarini ko'rasiz. Agar xato bo'lsa, qaysi test muvaffaqiyatsiz bo'lganini ko'rsatadi.

### ❇️ **Test sinovlari**:
- **Model testi**: `Task` modelining to'g'ri yaratilishi va `__str__` metodini sinaydi.
- **Serializer testi**: `TaskSerializer` maydonlari va `title` validatsiyasini tekshiradi.
- **API testi**: `/tasks/` va `/tasks/<pk>/` endpointlarining to'g'ri ishlashi, JWT autentifikatsiyasi, `IsOwnerOrReadOnly` ruxsati va sahifalashni sinaydi.
- **Signal testi**: Vazifa yaratilishi va o'chirilishi signallari log yozayotganini tekshiradi.

## ✅ 9. ADMIN PANELI ORQALI SINOV (IXTIYORIY)
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

## ✅ 10. SINOV UCHUN MASALALAR
📌 Testlarni sinash uchun quyidagi amallarni bajarib ko'ring:
1. `python manage.py test` ni ishga tushirib, barcha testlar muvaffaqiyatli o'tganini tekshiring.
2. `TaskSerializerTest` dagi `test_serializer_title_validation` testini sinab ko'rish uchun `title` ga 2 belgi kiriting va xato oling.
3. `TaskAPITest` dagi `test_update_task_owner_only` testini sinab ko'ring va boshqa foydalanuvchi `403 Forbidden` xatosini olishini tasdiqlang.
4. `TaskSignalTest` dagi loglarni tekshiring va Celery vazifasi sinxron tarzda ishlayotganini ko'ring.

