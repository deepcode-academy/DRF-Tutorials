# 🌐 DJANGO REST FRAMEWORK ASOSLARI

# 🧩 8-DARS: RUXSATLAR BILAN ISHLASH

Bu darsda Django REST Framework (DRF) yordamida ruxsatlar (permissions) ni sozlashni bosqichma-bosqich o'rganamiz. Ruxsatlar API resurslariga kirishni nazorat qiladi va foydalanuvchilarga faqat ruxsat berilgan amallarni bajarishga imkon beradi. Har bir qadam tushunarli va faqat ruxsatlarga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli va token autentifikatsiyasi asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan.
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- `TaskViewSet` `myapp/views.py` faylida sozlangan.
- Token autentifikatsiyasi va `/tasks/` endpointi `myproject/urls.py` faylida sozlangan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. RUXSATLARNI TUSHUNISH
📌 DRFda ruxsatlar (permissions) API endpointlariga kirishni nazorat qiladi. Ular foydalanuvchi autentifikatsiya qilinganligini yoki muayyan shartlarga javob berishini tekshiradi. DRFning o'rnatilgan ruxsat sinflari:
- `IsAuthenticated`: Faqat autentifikatsiya qilingan foydalanuvchilar uchun ruxsat.
- `IsAdminUser`: Faqat admin foydalanuvchilar uchun ruxsat.
- `IsAuthenticatedOrReadOnly`: Autentifikatsiya qilinganlar uchun yozish, boshqalar uchun faqat o'qish.
📌 Maxsus ruxsat sinflari ham yaratilishi mumkin.

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
📌 Yangilik sifatida `owner` maydoni qo'shildi, bu vazifaning egasini (foydalanuvchini) bog'laydi.

### ❇️ **Migratsiyalarni qo'llash**:
📌 `owner` maydoni qo'shilgani uchun migratsiyalarni yangilang:
```bash
python manage.py makemigrations
python manage.py migrate
```

### ❇️ **Serializer tekshirish**:
📌 `myapp/serializers.py` faylida `TaskSerializer` ni yangilaymiz:
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
📌 **Tushuntirish**:
- `owner`: Faqat o'qish uchun maydon sifatida qo'shildi, foydalanuvchi nomini ko'rsatadi.
- `read_only_fields`: `owner` foydalanuvchi tomonidan o'zgartirilmaydi.

## ✅ 4. MAXSUS RUXSAT SINFNI YARATISH
📌 Faqat vazifa egasi (owner) o'z vazifalarini tahrirlashi yoki o'chirishi mumkin bo'lgan maxsus ruxsat sinfini yaratamiz. `myapp/permissions.py` faylini yarating va quyidagi kodni qo'shing:
```python
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    def has_object_permission(self, request, view, obj):
        # GET, HEAD yoki OPTIONS so'rovlari uchun hamma ruxsat
        if request.method in permissions.SAFE_METHODS:
            return True
        # Yozish (POST, PUT, DELETE) uchun faqat egasi ruxsat
        return obj.owner == request.user
```
📌 **Tushuntirish**:
- `IsOwnerOrReadOnly`: GET (o'qish) so'rovlariga hamma ruxsat beriladi, lekin yozish (POST, PUT, DELETE) faqat vazifa egasi uchun mumkin.
- `has_object_permission`: Har bir obyekt uchun ruxsatni tekshiradi.

## ✅ 5. VIEWSET NI RUXSATLAR BILAN YANGILASH
📌 `myapp/views.py` faylida `TaskViewSet` ni yangilaymiz va maxsus ruxsat sinfini qo'shamiz:
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
        # Yangi vazifa yaratilganda owner ni so'rov yuborgan foydalanuvchi sifatida saqlaymiz
        serializer.save(owner=self.request.user)
```
📌 **Tushuntirish**:
- `permission_classes`: `IsAuthenticated` va `IsOwnerOrReadOnly` ruxsatlarini qo'llaydi.
- `perform_create`: Yangi vazifa yaratilganda `owner` maydoniga joriy foydalanuvchini avtomatik belgilaydi.

## ✅ 6. LOYIHA SOZLAMALARINI TEKSHIRISH
📌 `myproject/settings.py` faylida token autentifikatsiyasi sozlanganligiga ishonch hosil qiling:
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
}
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

## ✅ 8. RUXSATLARNI SINAB KO'RISH
📌 Serverni ishga tushiring:
```bash
python manage.py runserver
```

📌 API so'rovlarida tokenni `Authorization` sarlavhasida yuboring: `Token <token>`.

### ❇️ **Foydalanuvchi va token yaratish**:
📌 Ikkita foydalanuvchi yarating:
```bash
python manage.py createsuperuser  # username: user1, password: pass123
python manage.py createsuperuser  # username: user2, password: pass456
```
📌 Har bir foydalanuvchi uchun token oling:
```bash
curl -X POST http://127.0.0.1:8000/api-token-auth/ -d "username=user1&password=pass123"
curl -X POST http://127.0.0.1:8000/api-token-auth/ -d "username=user2&password=pass456"
```

### ❇️ **Create (Yangi vazifa qo'shish)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
📌 **Usul**: POST  
📌 **Masalan** (user1 uchun):
```bash
curl -X POST http://127.0.0.1:8000/tasks/ -H "Authorization: Token <user1_token>" -H "Content-Type: application/json" -d '{"title": "User1 vazifasi", "description": "Test vazifa"}'
```

### ❇️ **Read (Ma'lumotlarni o'qish)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
📌 **Usul**: GET  
📌 Har qanday autentifikatsiya qilingan foydalanuvchi barcha vazifalarni ko'rishi mumkin:
```bash
curl http://127.0.0.1:8000/tasks/ -H "Authorization: Token <user2_token>"
```

### ❇️ **Update (Ma'lumotlarni yangilash)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
📌 **Usul**: PATCH  
📌 **Masalan** (user1 vazifani yangilaydi):
```bash
curl -X PATCH http://127.0.0.1:8000/tasks/1/ -H "Authorization: Token <user1_token>" -H "Content-Type: application/json" -d '{"completed": true}'
```
📌 Agar user2 bu vazifani yangilamoqchi bo'lsa, `403 Forbidden` xatosi qaytadi:
```bash
curl -X PATCH http://127.0.0.1:8000/tasks/1/ -H "Authorization: Token <user2_token>" -H "Content-Type: application/json" -d '{"completed": true}'
```

### ❇️ **Delete (Ma'lumotlarni o'chirish)**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/1/`  
📌 **Usul**: DELETE  
📌 Faqat user1 o'chirishi mumkin:
```bash
curl -X DELETE http://127.0.0.1:8000/tasks/1/ -H "Authorization: Token <user1_token>"
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
📌 Ruxsatlarni sinash uchun quyidagi amallarni bajarib ko'ring:
1. `user1` bilan yangi vazifa qo'shing va `owner` maydonini tekshiring.
2. `user2` bilan `user1` ning vazifasini yangilashga urinib ko'ring va `403 Forbidden` xatosini oling.
3. `user2` bilan `/tasks/` manzilidan barcha vazifalarni ko'ring.
4. Tokensiz so'rov yuboring va `401 Unauthorized` xatosini oling.

