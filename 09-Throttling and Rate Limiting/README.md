# 🌐 DJANGO REST FRAMEWORK ASOSLARI

# 🧩 9-DARS: THROTTLING VA RATE LIMITING BILAN ISHLASH

Bu darsda Django REST Framework (DRF) yordamida throttling (so'rov cheklovi) va rate limiting (tezlikni cheklash) ni sozlashni bosqichma-bosqich o'rganamiz. Throttling API resurslarini ortiqcha ishlatishni oldini oladi va serverni himoya qiladi. Har bir qadam tushunarli va faqat throttling va rate limiting ga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli, token autentifikatsiyasi va ruxsatlar asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan (shu jumladan `owner` maydoni).
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- `TaskViewSet` `myapp/views.py` faylida sozlangan (autentifikatsiya va ruxsatlar bilan).
- Token autentifikatsiyasi va `/tasks/` endpointi `myproject/urls.py` faylida sozlangan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. THROTTLING VA RATE LIMITING NI TUSHUNISH
📌 **Throttling** (so'rov cheklovi) DRFda API ga keladigan so'rovlar sonini cheklash uchun ishlatiladi. Bu serverni haddan tashqari yuklanishdan himoya qiladi va xavfsizlikni oshiradi.
📌 **Rate Limiting**: Muayyan vaqt oralig'ida foydalanuvchi yoki IP manzilidan keladigan so'rovlar sonini cheklaydi.
📌 DRFning o'rnatilgan throttling sinflari:
- `AnonRateThrottle`: Autentifikatsiya qilinmagan (anonim) foydalanuvchilar uchun cheklov.
- `UserRateThrottle`: Autentifikatsiya qilingan foydalanuvchilar uchun cheklov.
- `ScopedRateThrottle`: Muayyan endpointlar uchun maxsus cheklovlar.

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

## ✅ 4. THROTTLING SOZLAMALARINI QO'SHISH
📌 DRFning o'rnatilgan throttling sinflarini ishlatish uchun `myproject/settings.py` faylini yangilaymiz:
```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework.authtoken',
    'myapp',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '10/hour',  # Anonim foydalanuvchilar uchun soatiga 10 ta so'rov
        'user': '20/hour',  # Autentifikatsiya qilingan foydalanuvchilar uchun soatiga 20 ta so'rov
    },
}
```
📌 **Tushuntirish**:
- `DEFAULT_THROTTLE_CLASSES`: Standart throttling sinflarini belgilaydi.
- `DEFAULT_THROTTLE_RATES`: Har bir sinf uchun cheklov stavkalarini sozlaydi (masalan, `10/hour` — soatiga 10 ta so'rov).
- `anon`: Autentifikatsiya qilinmagan foydalanuvchilar uchun.
- `user`: Autentifikatsiya qilingan foydalanuvchilar uchun.

## ✅ 5. VIEWSET NI TEKSHIRISH
📌 `myapp/views.py` faylida `TaskViewSet` oldingi darsdagidek qoladi, chunki throttling global sozlamalar orqali qo'llaniladi:
```python
from rest_framework import viewsets
from .models import Task
from .serializers import TaskSerializer
from .permissions import IsOwnerOrReadOnly
from rest_framework.permissions import IsAuthenticated

class TaskViewSet(viewsets.ModelViewSet):
    queryset = Task.objects.all()
    serializer_class = TaskSerializer
    permission_classes = [IsAuthenticated, IsOwnerOrReadOnly]

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```
📌 Throttling global sozlamalar orqali avtomatik qo'llaniladi, shuning uchun viewsetda qo'shimcha o'zgartirish talab qilinmaydi.

## ✅ 6. MAXSUS THROTTLE SINF YARATISH
📌 Muayyan endpoint uchun maxsus throttling sinfini yaratamiz. `myapp/throttles.py` faylini yarating va quyidagi kodni qo'shing:
```python
from rest_framework.throttling import UserRateThrottle

class CustomTaskThrottle(UserRateThrottle):
    scope = 'task-create'

    def get_rate(self):
        return '5/hour'  # Vazifa yaratish uchun soatiga 5 ta so'rov
```
📌 **Tushuntirish**:
- `CustomTaskThrottle`: `UserRateThrottle` dan meros oladi va faqat vazifa yaratish (POST) so'rovlari uchun cheklov qo'yadi.
- `scope`: Throttle sozlamalarida ishlatiladigan identifikator.

📌 `myproject/settings.py` faylida yangi scope ni qo'shing:
```python
REST_FRAMEWORK = {
    ...
    'DEFAULT_THROTTLE_RATES': {
        'anon': '10/hour',
        'user': '20/hour',
        'task-create': '5/hour',  # Yangi scope
    },
}
```

📌 `TaskViewSet` ni yangilaymiz va `POST` so'rovlariga maxsus throttle qo'shamiz:
```python
from rest_framework import viewsets
from .models import Task
from .serializers import TaskSerializer
from .permissions import IsOwnerOrReadOnly
from .throttles import CustomTaskThrottle
from rest_framework.permissions import IsAuthenticated

class TaskViewSet(viewsets.ModelViewSet):
    queryset = Task.objects.all()
    serializer_class = TaskSerializer
    permission_classes = [IsAuthenticated, IsOwnerOrReadOnly]
    throttle_classes = [CustomTaskThrottle]  # Maxsus throttle faqat POST uchun

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

## ✅ 7. URL SOZLASH
📌 `myproject/urls.py` faylida URL marshrutlari oldingi darsdagidek qoladi:
```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from rest_framework.authtoken.views import obtain_auth_token
from myapp.views import TaskViewSet

router = DefaultRouter()
router.register(r'tasks', TaskViewSet)

urlpatterns = [
    path('', include(router.urls)),
    path('api-token-auth/', obtain_auth_token, name='api_token_auth'),
]
```

## ✅ 8. THROTTLING NI SINAB KO'RISH
📌 Serverni ishga tushiring:
```bash
python manage.py runserver
```

📌 API so'rovlarida tokenni `Authorization` sarlavhasida yuboring: `Token <token>`.

### ❇️ **Foydalanuvchi va token yaratish**:
📌 Agar hali yaratilmagan bo'lsa, foydalanuvchi yarating:
```bash
python manage.py createsuperuser  # username: user1, password: pass123
```
📌 Token oling:
```bash
curl -X POST http://127.0.0.1:8000/api-token-auth/ -d "username=user1&password=pass123"
```

### ❇️ **Throttling sinovi (Yangi vazifa qo'shish)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
📌 **Usul**: POST  
📌 5 marta ketma-ket POST so'rov yuboring:
```bash
curl -X POST http://127.0.0.1:8000/tasks/ -H "Authorization: Token <user1_token>" -H "Content-Type: application/json" -d '{"title": "Test vazifa", "description": "Test"}'
```
📌 6-so'rovda `429 Too Many Requests` xatosi qaytadi, chunki `task-create` scope soatiga 5 ta so'rov bilan cheklangan.

### ❇️ **Read sinovi**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
📌 **Usul**: GET  
📌 20 marta ketma-ket GET so'rov yuboring:
```bash
curl http://127.0.0.1:8000/tasks/ -H "Authorization: Token <user1_token>"
```
📌 21-so'rovda `429 Too Many Requests` xatosi qaytadi, chunki `user` throttle soatiga 20 ta so'rov bilan cheklangan.

### ❇️ **Anonim foydalanuvchi sinovi**:
📌 Tokensiz GET so'rov yuboring:
```bash
curl http://127.0.0.1:8000/tasks/
```
📌 10 marta ketma-ket so'rov yuborsangiz, 11-so'rovda `429 Too Many Requests` xatosi qaytadi, chunki `anon` throttle soatiga 10 ta so'rov bilan cheklangan.

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
📌 Throttling ni sinash uchun quyidagi amallarni bajarib ko'ring:
1. `user1` bilan 5 ta POST so'rov yuboring va 6-so'rovda `429` xatosini oling.
2. `user1` bilan 20 ta GET so'rov yuboring va 21-so'rovda `429` xatosini oling.
3. Tokensiz 10 ta GET so'rov yuboring va 11-so'rovda `429` xatosini oling.
4. Admin panelida vazifa qo'shing va API orqali ko'ring.

