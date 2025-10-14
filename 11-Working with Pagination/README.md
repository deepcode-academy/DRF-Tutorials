# 🌐 DJANGO REST FRAMEWORK ASOSLARI

# 🧩 11-DARS: SAHIFALASH BILAN ISHLASH

Bu darsda Django REST Framework (DRF) yordamida sahifalash (pagination) ni sozlashni bosqichma-bosqich o'rganamiz. Sahifalash katta hajmdagi ma'lumotlarni kichik qismlarga bo'lib, API javoblarini optimallashtiradi va foydalanuvchi tajribasini yaxshilaydi. Har bir qadam tushunarli va faqat sahifalashga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli, token autentifikatsiyasi, ruxsatlar va filtrlash asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan (shu jumladan `owner` maydoni).
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- `TaskViewSet` `myapp/views.py` faylida sozlangan (autentifikatsiya, ruxsatlar va filtrlash bilan).
- Token autentifikatsiyasi va `/tasks/` endpointi `myproject/urls.py` faylida sozlangan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. SAHIFALASH NI TUSHUNISH
📌 **Sahifalash** (pagination) API javoblarida ma'lumotlarni kichik qismlarga bo'lib, har bir sahifada cheklangan miqdordagi ma'lumotni qaytaradi. Bu server yukini kamaytiradi va foydalanuvchiga ma'lumotlarni boshqarishni osonlashtiradi.
📌 DRFning o'rnatilgan sahifalash sinflari:
- `PageNumberPagination`: Sahifa raqami asosida sahifalash (masalan, ?page=2).
- `LimitOffsetPagination`: Belgilangan miqdor va ofset asosida sahifalash.
- `CursorPagination`: Kursor asosida sahifalash, katta ma'lumotlar uchun samarali.

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

## ✅ 4. SAHIFALASH SOZLAMALARINI QO'SHISH
📌 DRFning `PageNumberPagination` sinfidan foydalanamiz. `myproject/settings.py` faylini yangilaymiz:
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
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 5,  # Har bir sahifada 5 ta vazifa
}
```
📌 **Tushuntirish**:
- `DEFAULT_PAGINATION_CLASS`: Standart sahifalash sinfi sifatida `PageNumberPagination` ni belgilaydi.
- `PAGE_SIZE`: Har bir sahifada ko'rsatiladigan ma'lumotlar soni (5 ta vazifa).

## ✅ 5. MAXSUS SAHIFALASH SINF YARATISH
📌 Maxsus sahifalash sinfini yaratib, sahifalash javobini moslashtiramiz. `myapp/pagination.py` faylini yarating va quyidagi kodni qo'shing:
```python
from rest_framework.pagination import PageNumberPagination
from rest_framework.response import Response

class CustomTaskPagination(PageNumberPagination):
    page_size = 3  # Har bir sahifada 3 ta vazifa
    page_size_query_param = 'page_size'  # Foydalanuvchi sahifa hajmini o'zgartirishi mumkin
    max_page_size = 100  # Maksimal sahifa hajmi

    def get_paginated_response(self, data):
        return Response({
            'count': self.page.paginator.count,
            'next': self.get_next_link(),
            'previous': self.get_previous_link(),
            'results': data
        })
```
📌 **Tushuntirish**:
- `page_size`: Har bir sahifada 3 ta vazifa ko'rsatiladi.
- `page_size_query_param`: Foydalanuvchi `?page_size=10` kabi parametr yordamida sahifa hajmini o'zgartirishi mumkin.
- `max_page_size`: Sahifa hajmi 100 dan oshmaydi.
- `get_paginated_response`: Sahifalash javobini moslashtiradi (count, next, previous, results).

## ✅ 6. VIEWSET NI SAHIFALASH BILAN YANGILASH
📌 `myapp/views.py` faylida `TaskViewSet` ni yangilaymiz va maxsus sahifalash sinfini qo'shamiz:
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
    pagination_class = CustomTaskPagination  # Maxsus sahifalash sinfi

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

## ✅ 8. SAHIFALASH NI SINAB KO'RISH
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

### ❇️ **Standart sahifalash sinovi**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/`  
📌 **Usul**: GET  
📌 Har bir sahifada 3 ta vazifa ko'rinadi:
```bash
curl http://127.0.0.1:8000/tasks/ -H "Authorization: Token <user1_token>"
```
📌 Javob quyidagicha bo'ladi:
```json
{
    "count": 10,
    "next": "http://127.0.0.1:8000/tasks/?page=2",
    "previous": null,
    "results": [
        {...}, {...}, {...}  // 3 ta vazifa
    ]
}
```

### ❇️ **Muayyan sahifani ko'rish**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/?page=2`  
📌 **Usul**: GET  
```bash
curl http://127.0.0.1:8000/tasks/?page=2 -H "Authorization: Token <user1_token>"
```

### ❇️ **Sahifa hajmini o'zgartirish**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/?page_size=5`  
📌 **Usul**: GET  
📌 Har bir sahifada 5 ta vazifa ko'rinadi:
```bash
curl http://127.0.0.1:8000/tasks/?page_size=5 -H "Authorization: Token <user1_token>"
```

### ❇️ **Filtrlash bilan birgalikda sahifalash**:
📌 **Manzil**: `http://127.0.0.1:8000/tasks/?completed=true&page=1`  
📌 **Usul**: GET  
```bash
curl http://127.0.0.1:8000/tasks/?completed=true&page=1 -H "Authorization: Token <user1_token>"
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
📌 Sahifalashni sinash uchun quyidagi amallarni bajarib ko'ring:
1. `/tasks/` manzilida birinchi sahifani oling va 3 ta vazifa ekanligini tekshiring.
2. `/tasks/?page=2` manzilidan ikkinchi sahifani oling.
3. `/tasks/?page_size=5` manzilida sahifa hajmini 5 ga o'zgartiring.
4. `/tasks/?completed=true&page_size=2` manzilida faqat bajarilgan vazifalarni sahifalab ko'ring.

