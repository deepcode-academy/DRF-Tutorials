# 🌐 DJANGO REST FRAMEWORK ASOSLARI

# 🧩 7-DARS: AUTENTIFIKATSIYA (TOKEN-GA ASOSLANGAN AUTENTIFIKATSIYA)

Bu darsda Django REST Framework (DRF) yordamida token-ga asoslangan autentifikatsiyani sozlashni bosqichma-bosqich o'rganamiz. Token autentifikatsiyasi API ga xavfsiz kirishni ta'minlaydi. Har bir qadam tushunarli va faqat token autentifikatsiyasiga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan.
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- `TaskViewSet` `myapp/views.py` faylida sozlangan.
- `DefaultRouter` bilan `/tasks/` endpointi `myproject/urls.py` faylida ro'yxatdan o'tkazilgan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. TOKEN AUTENTIFIKATSIYASINI TUSHUNISH
📌 DRFning token-ga asoslangan autentifikatsiyasi har bir foydalanuvchiga noyob token beradi. Ushbu token API so'rovlarida foydalanuvchi identifikatsiyasi uchun ishlatiladi. Tokenlar HTTP sarlavhalarida (Authorization) yuboriladi.
📌 Afzalliklari:
- Oddiy va xavfsiz.
- Har bir foydalanuvchi uchun alohida token.
- Tokenni osongina o'chirish yoki yangilash mumkin.

## ✅ 3. TOKEN AUTENTIFIKATSIYASINI SOZLASH
📌 DRFning o'rnatilgan `TokenAuthentication` sinfidan foydalanamiz.

### ❇️ **DRF autentifikatsiya paketini o'rnatish**:
📌 Token autentifikatsiyasi uchun qo'shimcha paket o'rnatish shart emas, chunki u DRF bilan birga keladi. Ammo loyihada `rest_framework` o'rnatilganligini tekshiring:
```bash
pip install djangorestframework
```

### ❇️ **Token autentifikatsiyasini loyihaga qo'shish**:
📌 `myproject/settings.py` faylini yangilang va `REST_FRAMEWORK` sozlamalarini qo'shing:
```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework.authtoken',  # Token autentifikatsiyasi uchun
    'myapp',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',  # Faqat autentifikatsiya qilingan foydalanuvchilar uchun
    ],
}
```

### ❇️ **Migratsiyalarni qo'llash**:
📌 Token autentifikatsiyasi uchun ma'lumotlar bazasida `authtoken_token` jadvali yaratilishi kerak:
```bash
python manage.py migrate
```

## ✅ 4. VIEWSET NI AUTENTIFIKATSIYA BILAN YANGILASH
📌 `myapp/views.py` faylida `TaskViewSet` ni autentifikatsiya va ruxsatlar bilan yangilaymiz:
```python
from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticated
from .models import Task
from .serializers import TaskSerializer

class TaskViewSet(viewsets.ModelViewSet):
    queryset = Task.objects.all()
    serializer_class = TaskSerializer
    permission_classes = [IsAuthenticated]  # Faqat autentifikatsiya qilingan foydalanuvchilar
```
📌 **Tushuntirish**:
- `permission_classes = [IsAuthenticated]`: Faqat token bilan autentifikatsiya qilingan foydalanuvchilar API ga kira oladi.
- `TaskViewSet` oldingi darsda sozlangan bo'lib, CRUD operatsiyalarini qo'llab-quvvatlaydi.

## ✅ 5. ROUTER BILAN URL SOZLASH
📌 `myproject/urls.py` faylida URL marshrutlari oldingi darsdagidek qoladi, lekin token olish uchun qo'shimcha endpoint qo'shamiz:
```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from rest_framework.authtoken.views import obtain_auth_token
from myapp.views import TaskViewSet

router = DefaultRouter()
router.register(r'tasks', TaskViewSet)

urlpatterns = [
    path('', include(router.urls)),
    path('api-token-auth/', obtain_auth_token, name='api_token_auth'),  # Token olish uchun
]
```
📌 **Tushuntirish**:
- `obtain_auth_token`: Foydalanuvchi login va parol yordamida token olishi uchun endpoint.
- `/tasks/` va `/tasks/<pk>/` endpointlari `TaskViewSet` uchun ishlaydi.

## ✅ 6. FOYDALANUVCHI VA TOKEN YARATISH
📌 Token autentifikatsiyasi uchun foydalanuvchi va token yaratamiz.

### ❇️ **Foydalanuvchi yaratish**:
📌 Admin paneli orqali yoki buyruq satri yordamida foydalanuvchi yarating:
```bash
python manage.py createsuperuser
```
📌 Masalan: `username: testuser`, `password: testpass123`.

### ❇️ **Token yaratish**:
📌 Token olish uchun `/api-token-auth/` endpointiga POST so'rov yuboring:
```bash
curl -X POST http://127.0.0.1:8000/api-token-auth/ -d "username=testuser&password=testpass123"
```
📌 Javob sifatida token qaytadi, masalan:
```json
{"token": "9944b09199c62bcf9418ad846dd0e4bbafc6ee0b"}
```
📌 Ushbu tokenni saqlang, chunki u API so'rovlarida ishlatiladi.

## ✅ 7. AUTENTIFIKATSIYA BILAN API NI SINAB KO'RISH
📌 Serverni ishga tushiring:
```bash
python manage.py runserver
```

📌 API so'rovlarida tokenni `Authorization` sarlavhasida yuboring: `Token <token>`.

### ❇️ **Create (Yangi vazifa qo'shish)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
📌 **Usul**: POST  
📌 **Masalan** (Postman orqali):
```bash
curl -X POST http://127.0.0.1:8000/tasks/ -H "Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbafc6ee0b" -H "Content-Type: application/json" -d '{"title": "Yangi vazifa", "description": "Bu test vazifa", "completed": false}'
```
📌 Token bo'lmasa, `401 Unauthorized` xatosi qaytadi.

### ❇️ **Read (Ma'lumotlarni o'qish)**:
- **Barcha vazifalarni ko'rish**:  
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
  📌 **Usul**: GET  
  📌 Masalan:
  ```bash
  curl http://127.0.0.1:8000/tasks/ -H "Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbafc6ee0b"
  ```

- **Bitta vazifani ko'rish**:  
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
  📌 **Usul**: GET  
  📌 Masalan:
  ```bash
  curl http://127.0.0.1:8000/tasks/1/ -H "Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbafc6ee0b"
  ```

### ❇️ **Update (Ma'lumotlarni yangilash)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
📌 **Usul**: PATCH  
📌 **Masalan**:
```bash
curl -X PATCH http://127.0.0.1:8000/tasks/1/ -H "Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbafc6ee0b" -H "Content-Type: application/json" -d '{"completed": true}'
```

### ❇️ **Delete (Ma'lumotlarni o'chirish)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
📌 **Usul**: DELETE  
📌 **Masalan**:
```bash
curl -X DELETE http://127.0.0.1:8000/tasks/1/ -H "Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbafc6ee0b"
```

## ✅ 8. ADMIN PANELI ORQALI SINOV (IXTIYORIY)
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

## ✅ 9. SINOV UCHUN MASALALAR
📌 Token autentifikatsiyasini sinash uchun quyidagi amallarni bajarib ko'ring:
1. `/api-token-auth/` manzilidan token oling.
2. Token bilan `/tasks/` manzilida yangi vazifa qo'shing.
3. Tokensiz so'rov yuboring va `401 Unauthorized` xatosini oling.
4. `/tasks/1/` manzilida vazifani yangilang yoki o'chiring.
