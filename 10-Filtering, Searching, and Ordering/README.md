# 🌐 DJANGO REST FRAMEWORK ASOSLARI

# 🧩 10-DARS: FILTRLASH, QIDIRISH VA TARTIBLASH

Bu darsda Django REST Framework (DRF) yordamida ma'lumotlarni filtrlash, qidirish va tartiblash funksiyalarini sozlashni bosqichma-bosqich o'rganamiz. Bu funksiyalar API dan olingan ma'lumotlarni foydalanuvchi ehtiyojlariga moslashtirishga yordam beradi. Har bir qadam tushunarli va faqat filtrlash, qidirish va tartiblashga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli, token autentifikatsiyasi va ruxsatlar asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan (shu jumladan `owner` maydoni).
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- `TaskViewSet` `myapp/views.py` faylida sozlangan (autentifikatsiya va ruxsatlar bilan).
- Token autentifikatsiyasi va `/tasks/` endpointi `myproject/urls.py` faylida sozlangan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. FILTRLASH, QIDIRISH VA TARTIBLASH NI TUSHUNISH
📌 **Filtrlash**: Ma'lumotlarni muayyan shartlar asosida cheklash (masalan, faqat bajarilgan vazifalarni ko'rsatish).
📌 **Qidirish**: Ma'lumotlarni kalit so'z bo'yicha izlash (masalan, sarlavhada muayyan so'zni qidirish).
📌 **Tartiblash**: Ma'lumotlarni muayyan maydon bo'yicha tartiblash (masalan, yaratilgan sana bo'yicha).
📌 DRF bu funksiyalarni `django-filter`, `SearchFilter` va `OrderingFilter` kabi vositalar orqali qo'llab-quvvatlaydi.

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

## ✅ 4. FILTRLASH UCHUN DJANGO-FILTER O'RNATISH
📌 Filtrlash uchun `django-filter` paketini o'rnatamiz:
```bash
pip install django-filter
```

📌 `myproject/settings.py` faylida `django-filter` ni qo'shing:
```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework.authtoken',
    'django_filters',
    'myapp',
]
```

## ✅ 5. VIEWSET NI FILTRLASH, QIDIRISH VA TARTIBLASH BILAN YANGILASH
📌 `myapp/views.py` faylida `TaskViewSet` ni yangilaymiz va filtrlash, qidirish va tartiblash funksiyalarini qo'shamiz:
```python
from rest_framework import viewsets
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.filters import SearchFilter, OrderingFilter
from .models import Task
from .serializers import TaskSerializer
from .permissions import IsOwnerOrReadOnly
from rest_framework.permissions import IsAuthenticated

class TaskViewSet(viewsets.ModelViewSet):
    queryset = Task.objects.all()
    serializer_class = TaskSerializer
    permission_classes = [IsAuthenticated, IsOwnerOrReadOnly]
    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
    filterset_fields = ['completed', 'owner']  # Filtrlash uchun maydonlar
    search_fields = ['title', 'description']   # Qidirish uchun maydonlar
    ordering_fields = ['created_at', 'completed']  # Tartiblash uchun maydonlar
    ordering = ['created_at']  # Standart tartiblash

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```
📌 **Tushuntirish**:
- `filter_backends`: `DjangoFilterBackend`, `SearchFilter` va `OrderingFilter` ni qo'shadi.
- `filterset_fields`: `completed` va `owner` bo'yicha filtrlash imkonini beradi.
- `search_fields`: `title` va `description` maydonlarida qidirish imkonini beradi.
- `ordering_fields`: `created_at` va `completed` bo'yicha tartiblash imkonini beradi.
- `ordering`: Standart tartiblash `created_at` bo'yicha.

## ✅ 6. LOYIHA SOZLAMALARINI YANGILASH
📌 `myproject/settings.py` faylida `REST_FRAMEWORK` sozlamalarini yangilang:
```python
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
        'anon': '10/hour',
        'user': '20/hour',
    },
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
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

## ✅ 8. FILTRLASH, QIDIRISH VA TARTIBLASH NI SINAB KO'RISH
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

### ❇️ **Filtrlash sinovi**:
- **Faqat bajarilgan vazifalarni ko'rish**:
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/?completed=true`  
  📌 **Usul**: GET  
  📌 Masalan:
  ```bash
  curl http://127.0.0.1:8000/tasks/?completed=true -H "Authorization: Token <user1_token>"
  ```
- **Muayyan foydalanuvchi vazifalarini ko'rish**:
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/?owner=1` (1 — foydalanuvchi ID si)  
  📌 **Usul**: GET  
  📌 Masalan:
  ```bash
  curl http://127.0.0.1:8000/tasks/?owner=1 -H "Authorization: Token <user1_token>"
  ```

### ❇️ **Qidirish sinovi**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/?search=test`  
📌 **Usul**: GET  
📌 `title` yoki `description` da "test" so'zini qidiradi:
```bash
curl http://127.0.0.1:8000/tasks/?search=test -H "Authorization: Token <user1_token>"
```

### ❇️ **Tartiblash sinovi**:
- **Yaratilgan sana bo'yicha teskari tartiblash**:
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/?ordering=-created_at`  
  📌 **Usul**: GET  
  📌 Masalan:
  ```bash
  curl http://127.0.0.1:8000/tasks/?ordering=-created_at -H "Authorization: Token <user1_token>"
  ```
- **Bajarilgan holati bo'yicha tartiblash**:
  📌 **Manzil**: `http://127.0.0.1:8000/tasks/?ordering=completed`  
  📌 **Usul**: GET  
  ```bash
  curl http://127.0.0.1:8000/tasks/?ordering=completed -H "Authorization: Token <user1_token>"
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
📌 Filtrlash, qidirish va tartiblashni sinash uchun quyidagi amallarni bajarib ko'ring:
1. `/tasks/?completed=true` manzilidan faqat bajarilgan vazifalarni oling.
2. `/tasks/?search=vazifa` manzilidan "vazifa" so'zini o'z ichiga olgan vazifalarni qidiring.
3. `/tasks/?ordering=-created_at` manzilidan eng so'nggi vazifalarni birinchi ko'ring.
4. Admin panelida bir nechta vazifa qo'shing va filtrlash/qidirishni sinab ko'ring.
