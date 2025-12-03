# 🧩 12-DARS: JWT AUTENTIFIKATSIYASI BILAN ISHLASH

Bu darsda Django REST Framework (DRF) yordamida JSON Web Token (JWT) autentifikatsiyasini sozlashni bosqichma-bosqich o'rganamiz. JWT autentifikatsiyasi token-ga asoslangan autentifikatsiyaning zamonaviy va xavfsiz shakli bo'lib, foydalanuvchi ma'lumotlarini token ichida saqlaydi. Har bir qadam tushunarli va faqat JWT autentifikatsiyasiga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli, ruxsatlar va sahifalash asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan (shu jumladan `owner` maydoni).
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- `TaskViewSet` `myapp/views.py` faylida sozlangan (ruxsatlar, filtrlash va sahifalash bilan).
- `/tasks/` endpointi `myproject/urls.py` faylida sozlangan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. JWT AUTENTIFIKATSIYASINI TUSHUNISH
📌 **JWT** (JSON Web Token) uch qismdan iborat: **Header**, **Payload** va **Signature**. U foydalanuvchi ma'lumotlarini (masalan, user ID) token ichida shifrlangan holda saqlaydi va server uni tekshiradi.
📌 Afzalliklari:
- Stateless: Serverda token saqlanmaydi, bu esa server resurslarini tejaydi.
- Xavfsiz: Token imzo orqali himoyalanadi.
- Moslashuvchan: Access va refresh tokenlar orqali qulay autentifikatsiya.
📌 DRFda JWT sozlash uchun `djangorestframework-simplejwt` paketidan foydalanamiz.

## ✅ 3. JWT PAKETINI O'RNATISH
📌 `djangorestframework-simplejwt` paketini o'rnatamiz:
```bash
pip install djangorestframework-simplejwt
```

## ✅ 4. LOYIHA SOZLAMALARINI YANGILASH
📌 `myproject/settings.py` faylida JWT autentifikatsiyasini qo'shamiz:
```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework.authtoken',  # Oldingi token autentifikatsiyasi uchun (ixtiyoriy)
    'django_filters',
    'myapp',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',  # JWT autentifikatsiyasi
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
```

## ✅ 5. JWT URL LARINI SOZLASH
📌 `myproject/urls.py` faylini yangilaymiz va JWT token olish/yangilash endpointlarini qo'shamiz:
```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from myapp.views import TaskViewSet
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView

router = DefaultRouter()
router.register(r'tasks', TaskViewSet)

urlpatterns = [
    path('', include(router.urls)),
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),  # Access va refresh token olish
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),  # Refresh token yordamida yangi access token olish
]
```
📌 **Tushuntirish**:
- `TokenObtainPairView`: Foydalanuvchi login/parol orqali access va refresh tokenlarini olish uchun.
- `TokenRefreshView`: Refresh token yordamida yangi access token olish uchun.

## ✅ 6. MODEL VA SERIALIZERNI TEKSHIRISH
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

## ✅ 7. VIEWSET NI TEKSHIRISH
📌 `myapp/views.py` faylida `TaskViewSet` oldingi darsdagidek qoladi, chunki JWT autentifikatsiyasi global sozlamalar orqali qo'llaniladi:
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

## ✅ 8. JWT AUTENTIFIKATSIYASINI SINAB KO'RISH
📌 Serverni ishga tushiring:
```bash
python manage.py runserver
```

### ❇️ **Foydalanuvchi yaratish**:
📌 Agar hali yaratilmagan bo'lsa, foydalanuvchi yarating:
```bash
python manage.py createsuperuser  # username: user1, password: pass123
```

### ❇️ **JWT token olish**:
📌 **Manzil**: `http://127.0.0.1:8000/api/token/`  
📌 **Usul**: POST  
📌 Access va refresh tokenlarni olish:
```bash
curl -X POST http://127.0.0.1:8000/api/token/ -H "Content-Type: application/json" -d '{"username": "user1", "password": "pass123"}'
```
📌 Javob quyidagicha bo'ladi:
```json
{
    "refresh": "<refresh_token>",
    "access": "<access_token>"
}
```
📌 `access_token` ni saqlang, chunki u API so'rovlarida ishlatiladi. `refresh_token` esa yangi access token olish uchun ishlatiladi.

### ❇️ **API so'rovlarida JWT ishlatish**:
📌 API so'rovlarida `Authorization` sarlavhasida `Bearer <access_token>` yuboring.

- **Barcha vazifalarni ko'rish**:
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
  📌 **Usul**: GET  
  ```bash
  curl http://127.0.0.1:8000/tasks/ -H "Authorization: Bearer <access_token>"
  ```

- **Yangi vazifa qo'shish**:
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
  📌 **Usul**: POST  
  ```bash
  curl -X POST http://127.0.0.1:8000/tasks/ -H "Authorization: Bearer <access_token>" -H "Content-Type: application/json" -d '{"title": "Yangi vazifa", "description": "Test vazifa"}'
  ```

- **Vazifani yangilash**:
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
  📌 **Usul**: PATCH  
  ```bash
  curl -X PATCH http://127.0.0.1:8000/tasks/1/ -H "Authorization: Bearer <access_token>" -H "Content-Type: application/json" -d '{"completed": true}'
  ```

- **Vazifani o'chirish**:
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
  📌 **Usul**: DELETE  
  ```bash
  curl -X DELETE http://127.0.0.1:8000/tasks/1/ -H "Authorization: Bearer <access_token>"
  ```

### ❇️ **Refresh token bilan yangi access token olish**:
📌 **Manzil**: `http://127.0.0.1:8000/api/token/refresh/`  
📌 **Usul**: POST  
```bash
curl -X POST http://127.0.0.1:8000/api/token/refresh/ -H "Content-Type: application/json" -d '{"refresh": "<refresh_token>"}'
```
📌 Javobda yangi `access_token` qaytadi.

### ❇️ **Tokensiz so'rov sinovi**:
📌 Tokensiz so'rov yuborsangiz, `401 Unauthorized` xatosi qaytadi:
```bash
curl http://127.0.0.1:8000/tasks/
```

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
📌 JWT autentifikatsiyasini sinash uchun quyidagi amallarni bajarib ko'ring:
1. `/api/token/` manzilidan access va refresh tokenlarni oling.
2. Access token bilan `/tasks/` manzilida yangi vazifa qo'shing.
3. Refresh token yordamida yangi access token oling va u bilan so'rov yuboring.
4. Tokensiz so'rov yuboring va `401 Unauthorized` xatosini oling.

