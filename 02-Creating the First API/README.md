# 🧩 2-DARS: BIRINCHI API NI YARATISH

## 🎯 Dars Maqsadi

Bu darsda siz Django REST Framework yordamida to'liq ishlaydigan RESTful API yaratishni o'rganasiz. To-Do List API yaratish orqali barcha CRUD operatsiyalarini amaliyotda ko'rasiz.

**Dars oxirida siz:**
- ✅ Model yaratish va ma'nosi
- ✅ Serializer yaratish va ularning roli
- ✅ Function-based va Class-based views farqi
- ✅ API endpoint yaratish usullari
- ✅ DRF browsable API bilan ishlash
- ✅ Postman va cURL orqali API test qilish

---

## 📚 Oldingi Darsdan Kerakli Bilimlar

Bu darsni boshlashdan oldin quyidagilar tayyor bo'lishi kerak:

- [x] Virtual environment faollashgan
- [x] Django va DRF o'rnatilgan
- [x] Django loyihasi yaratilgan (`myproject`)
- [x] DRF `INSTALLED_APPS` da ro'yxatdan o'tgan

> **Eslatma:** Agar yuqoridagilar tayyor bo'lmasa, 1-darsga qayting!

---

## 🚀 1. LOYIHANI TAYYORLASH

### 1.1 Yangi Ilova Yaratish

To-Do API uchun alohida ilova yaratamiz:

```bash
python manage.py startapp tasks
```

### 1.2 Ilovani Ro'yxatdan O'tkazish

`myproject/settings.py` faylida:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Third-party apps
    'rest_framework',
    
    # Local apps
    'tasks',  # Yangi ilova
]
```

---

## 💾 2. MODEL YARATISH

### 2.1 Task Modeli

`tasks/models.py` faylida:

```python
from django.db import models
from django.contrib.auth.models import User

class Task(models.Model):
    """
    Vazifalar (To-Do) modeli
    """
    # Vazifa kategoriyalari
    PRIORITY_CHOICES = [
        ('low', 'Past'),
        ('medium', 'O\'rta'),
        ('high', 'Yuqori'),
    ]
    
    title = models.CharField(
        max_length=200,
        verbose_name="Sarlavha",
        help_text="Vazifa sarlavhasi"
    )
    description = models.TextField(
        blank=True,
        null=True,
        verbose_name="Tavsif",
        help_text="Vazifa haqida batafsil ma'lumot"
    )
    completed = models.BooleanField(
        default=False,
        verbose_name="Bajarilganmi?",
        help_text="Vazifa bajarilgan yoki bajarilmagan"
    )
    priority = models.CharField(
        max_length=10,
        choices=PRIORITY_CHOICES,
        default='medium',
        verbose_name="Muhimlik darajasi"
    )
    due_date = models.DateField(
        blank=True,
        null=True,
        verbose_name="Tugash sanasi"
    )
    created_at = models.DateTimeField(
        auto_now_add=True,
        verbose_name="Yaratilgan vaqt"
    )
    updated_at = models.DateTimeField(
        auto_now=True,
        verbose_name="Yangilangan vaqt"
    )
    
    class Meta:
        verbose_name = "Vazifa"
        verbose_name_plural = "Vazifalar"
        ordering = ['-created_at']  # Eng yangilari birinchi
        
    def __str__(self):
        return f"{self.title} - {'✅' if self.completed else '⏳'}"
```

**Tushuntirishlar:**

| Field | Turi | Vazifasi |
|-------|------|----------|
| `title` | CharField | Vazifa nomi (majburiy) |
| `description` | TextField | Batafsil tavsif (ixtiyoriy) |
| `completed` | BooleanField | Bajarilgan/Bajarilmagan holat |
| `priority` | CharField | Muhimlik darajasi (choices bilan) |
| `due_date` | DateField | Tugash sanasi |
| `created_at` | DateTimeField | Avtomatik yaratilish vaqti |
| `updated_at` | DateTimeField | Avtomatik yangilanish vaqti |

**Choices nima?**
- Ma'lum qiymatlardan birini tanlash imkoniyati
- Frontend uchun dropdown yaratadi
- Database da faqat ruxsat etilgan qiymatlar saqlanadi

### 2.2 Migratsiyalar

```bash
# Migratsiya fayllarini yaratish
python manage.py makemigrations tasks

# Migratsiyalarni qo'llash
python manage.py migrate
```

**Natija:**
```
Migrations for 'tasks':
  tasks/migrations/0001_initial.py
    - Create model Task
```

---

## 🔧 3. SERIALIZER YARATISH

### 3.1 Serializer Nima?

**Serializer** - ma'lumotlarni turli formatlar orasida o'tkazuvchi vosita:

```
Python Object (Model) ←→ JSON ←→ Python Dict
```

### 3.2 Basic Serializer

`tasks/serializers.py` faylini yarating:

```python
from rest_framework import serializers
from .models import Task

class TaskSerializer(serializers.ModelSerializer):
    """
    Task modelini JSON formatga aylantirish uchun serializer
    """
    # Read-only field - faqat response da ko'rinadi
    days_since_created = serializers.SerializerMethodField()
    
    class Meta:
        model = Task
        fields = [
            'id',
            'title',
            'description',
            'completed',
            'priority',
            'due_date',
            'created_at',
            'updated_at',
            'days_since_created',
        ]
        read_only_fields = ['id', 'created_at', 'updated_at']
    
    def get_days_since_created(self, obj):
        """
        Vazifa yaratilganiga necha kun bo'lganini hisoblash
        """
        from datetime import datetime
        delta = datetime.now().date() - obj.created_at.date()
        return delta.days
    
    def validate_title(self, value):
        """
        Title validation - bo'sh bo'lmasligi kerak
        """
        if len(value.strip()) < 3:
            raise serializers.ValidationError(
                "Sarlavha kamida 3 ta belgidan iborat bo'lishi kerak!"
            )
        return value
    
    def validate(self, data):
        """
        Umumiy validation - bir nechta fieldlarni birga tekshirish
        """
        # Agar due_date o'tmishda bo'lsa, xato berish
        from datetime import datetime
        if data.get('due_date') and data['due_date'] < datetime.now().date():
            raise serializers.ValidationError({
                'due_date': "Tugash sanasi o'tmishda bo'lishi mumkin emas!"
            })
        return data
```

**Asosiy Komponentlar:**

1. **`fields`** - JSON da qaysi fieldlar ko'rinadi
2. **`read_only_fields`** - Faqat o'qish uchun (POST/PUT da o'zgartirish mumkin emas)
3. **`SerializerMethodField`** - Custom field yaratish
4. **`validate_<field_name>`** - Bitta fieldni tekshirish
5. **`validate`** - Barcha fieldlarni birga tekshirish

### 3.3 Serializer Variations

**Oddiy Serializer (Barcha fieldlar):**

```python
class TaskListSerializer(serializers.ModelSerializer):
    """
    Ro'yxat ko'rinishi uchun (kam ma'lumot)
    """
    class Meta:
        model = Task
        fields = ['id', 'title', 'completed', 'priority']
```

**Batafsil Serializer:**

```python
class TaskDetailSerializer(serializers.ModelSerializer):
    """
    Batafsil ko'rinish uchun (barcha ma'lumot)
    """
    class Meta:
        model = Task
        fields = '__all__'  # Barcha fieldlar
```

---

## 🎨 4. VIEWS YARATISH

### 4.1 Function-Based Views (FBV)

`tasks/views.py` da:

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from .models import Task
from .serializers import TaskSerializer

@api_view(['GET', 'POST'])
def task_list(request):
    """
    Barcha vazifalarni olish (GET) yoki yangi vazifa yaratish (POST)
    """
    if request.method == 'GET':
        # Barcha vazifalarni olish
        tasks = Task.objects.all()
        serializer = TaskSerializer(tasks, many=True)
        return Response(serializer.data)
    
    elif request.method == 'POST':
        # Yangi vazifa yaratish
        serializer = TaskSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


@api_view(['GET', 'PUT', 'PATCH', 'DELETE'])
def task_detail(request, pk):
    """
    Bitta vazifani olish (GET), yangilash (PUT/PATCH) yoki o'chirish (DELETE)
    """
    try:
        task = Task.objects.get(pk=pk)
    except Task.DoesNotExist:
        return Response(
            {'error': 'Vazifa topilmadi!'},
            status=status.HTTP_404_NOT_FOUND
        )
    
    if request.method == 'GET':
        serializer = TaskSerializer(task)
        return Response(serializer.data)
    
    elif request.method == 'PUT' or request.method == 'PATCH':
        partial = request.method == 'PATCH'  # PATCH - qisman yangilash
        serializer = TaskSerializer(task, data=request.data, partial=partial)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    elif request.method == 'DELETE':
        task.delete()
        return Response(
            {'message': 'Vazifa o\'chirildi!'},
            status=status.HTTP_204_NO_CONTENT
        )
```

**FBV URL Configuration:**

`tasks/urls.py` yarating:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('tasks/', views.task_list, name='task-list'),
    path('tasks/<int:pk>/', views.task_detail, name='task-detail'),
]
```

### 4.2 Class-Based Views (CBV) - APIView

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from .models import Task
from .serializers import TaskSerializer

class TaskListAPIView(APIView):
    """
    Barcha vazifalar bilan ishlash
    """
    def get(self, request):
        """Barcha vazifalarni olish"""
        tasks = Task.objects.all()
        serializer = TaskSerializer(tasks, many=True)
        return Response(serializer.data)
    
    def post(self, request):
        """Yangi vazifa yaratish"""
        serializer = TaskSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


class TaskDetailAPIView(APIView):
    """
    Bitta vazifa bilan ishlash
    """
    def get_object(self, pk):
        """Vazifani topish"""
        try:
            return Task.objects.get(pk=pk)
        except Task.DoesNotExist:
            return None
    
    def get(self, request, pk):
        """Bitta vazifani olish"""
        task = self.get_object(pk)
        if task is None:
            return Response(
                {'error': 'Topilmadi'},
                status=status.HTTP_404_NOT_FOUND
            )
        serializer = TaskSerializer(task)
        return Response(serializer.data)
    
    def put(self, request, pk):
        """Vazifani yangilash (to'liq)"""
        task = self.get_object(pk)
        if task is None:
            return Response(
                {'error': 'Topilmadi'},
                status=status.HTTP_404_NOT_FOUND
            )
        serializer = TaskSerializer(task, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    def patch(self, request, pk):
        """Vazifani yangilash (qisman)"""
        task = self.get_object(pk)
        if task is None:
            return Response(
                {'error': 'Topilmadi'},
                status=status.HTTP_404_NOT_FOUND
            )
        serializer = TaskSerializer(task, data=request.data, partial=True)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    def delete(self, request, pk):
        """Vazifani o'chirish"""
        task = self.get_object(pk)
        if task is None:
            return Response(
                {'error': 'Topilmadi'},
                status=status.HTTP_404_NOT_FOUND
            )
        task.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### 4.3 ViewSet (Eng Professional Usul)

```python
from rest_framework import viewsets
from rest_framework.decorators import action
from rest_framework.response import Response
from .models import Task
from .serializers import TaskSerializer

class TaskViewSet(viewsets.ModelViewSet):
    """
    Task CRUD API - Avtomatik barcha operatsiyalar
    """
    queryset = Task.objects.all()
    serializer_class = TaskSerializer
    
    # Custom action - bajarilgan vazifalarni olish
    @action(detail=False, methods=['get'])
    def completed(self, request):
        """
        URL: /tasks/completed/
        Faqat bajarilgan vazifalarni qaytaradi
        """
        completed_tasks = Task.objects.filter(completed=True)
        serializer = self.get_serializer(completed_tasks, many=True)
        return Response(serializer.data)
    
    # Custom action - vazifani bajarilgan deb belgilash
    @action(detail=True, methods=['post'])
    def mark_complete(self, request, pk=None):
        """
        URL: /tasks/{id}/mark_complete/
        Vazifani bajarilgan deb belgilaydi
        """
        task = self.get_object()
        task.completed = True
        task.save()
        serializer = self.get_serializer(task)
        return Response(serializer.data)
    
    @action(detail=False, methods=['get'])
    def high_priority(self, request):
        """
        URL: /tasks/high_priority/
        Yuqori muhimlikdagi vazifalarni qaytaradi
        """
        high_tasks = Task.objects.filter(priority='high', completed=False)
        serializer = self.get_serializer(high_tasks, many=True)
        return Response(serializer.data)
```

**ViewSet URL Configuration:**

`myproject/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from tasks.views import TaskViewSet

router = DefaultRouter()
router.register(r'tasks', TaskViewSet, basename='task')

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include(router.urls)),
]
```

---

## 🌐 5. URL ROUTING

### 5.1 ViewSet Routing (Tavsiya Etiladi)

**Avtomatik yaratilgan URL'lar:**

| HTTP Method | URL | Action | Tavsif |
|-------------|-----|--------|--------|
| GET | `/api/tasks/` | list | Barcha vazifalar |
| POST | `/api/tasks/` | create | Yangi vazifa |
| GET | `/api/tasks/{id}/` | retrieve | Bitta vazifa |
| PUT | `/api/tasks/{id}/` | update | To'liq yangilash |
| PATCH | `/api/tasks/{id}/` | partial_update | Qisman yangilash |
| DELETE | `/api/tasks/{id}/` | destroy | O'chirish |
| GET | `/api/tasks/completed/` | completed | Custom action |
| POST | `/api/tasks/{id}/mark_complete/` | mark_complete | Custom action |
| GET | `/api/tasks/high_priority/` | high_priority | Custom action |

---

## 🛡️ 6. ADMIN PANEL SOZLASH

`tasks/admin.py`:

```python
from django.contrib import admin
from .models import Task

@admin.register(Task)
class TaskAdmin(admin.ModelAdmin):
    """
    Task modelini admin panelda boshqarish
    """
    list_display = [
        'id',
        'title',
        'completed',
        'priority',
        'due_date',
        'created_at'
    ]
    list_filter = ['completed', 'priority', 'created_at']
    search_fields = ['title', 'description']
    list_editable = ['completed', 'priority']  # Ro'yxatda o'zgartirish
    readonly_fields = ['created_at', 'updated_at']
    date_hierarchy = 'created_at'
    
    # Fieldlar guruhlanishi
    fieldsets = (
        ('Asosiy Ma\'lumotlar', {
            'fields': ('title', 'description')
        }),
        ('Holat', {
            'fields': ('completed', 'priority', 'due_date')
        }),
        ('Vaqt Ma\'lumotlari', {
            'fields': ('created_at', 'updated_at'),
            'classes': ('collapse',)  # Yopilgan holatda
        }),
    )
```

**Admin uchun superuser yaratish:**

```bash
python manage.py createsuperuser
```

---

## ✅ 7. API NI TEST QILISH

### 7.1 DRF Browsable API

**Server ishga tushiring:**
```bash
python manage.py runserver
```

**Manzillar:**
- `http://127.0.0.1:8000/api/tasks/` - Barcha vazifalar
- `http://127.0.0.1:8000/api/tasks/1/` - 1-vazifa
- `http://127.0.0.1:8000/api/tasks/completed/` - Bajarilgan vazifalar

### 7.2 cURL orqali Test

**GET - Barcha vazifalar:**
```bash
curl http://127.0.0.1:8000/api/tasks/
```

**POST - Yangi vazifa:**
```bash
curl -X POST http://127.0.0.1:8000/api/tasks/ \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Django o'\''rganish",
    "description": "DRF kursini tugatish",
    "priority": "high",
    "due_date": "2026-02-01"
  }'
```

**PATCH - Bajarilgan deb belgilash:**
```bash
curl -X PATCH http://127.0.0.1:8000/api/tasks/1/ \
  -H "Content-Type: application/json" \
  -d '{"completed": true}'
```

**DELETE - O'chirish:**
```bash
curl -X DELETE http://127.0.0.1:8000/api/tasks/1/
```

### 7.3 Postman orqali Test

**Import qilish uchun collection:**

1. Postman ochish
2. New Collection yaratish: "Task API"
3. Requestlar qo'shish:

**GET All Tasks:**
- Method: GET
- URL: `http://127.0.0.1:8000/api/tasks/`

**Create Task:**
- Method: POST
- URL: `http://127.0.0.1:8000/api/tasks/`
- Body (JSON):
```json
{
    "title": "Test vazifa",
    "description": "Postman orqali yaratilgan",
    "priority": "medium"
}
```

---

## 📊 8. RESPONSE FORMAT

### 8.1 List Response

```json
[
    {
        "id": 1,
        "title": "Django o'rganish",
        "description": "DRF kursini tugatish",
        "completed": false,
        "priority": "high",
        "due_date": "2026-02-01",
        "created_at": "2026-01-21T10:30:00Z",
        "updated_at": "2026-01-21T10:30:00Z",
        "days_since_created": 0
    },
    {
        "id": 2,
        "title": "API yaratish",
        "description": "To-Do API loyihasi",
        "completed": true,
        "priority": "medium",
        "due_date": null,
        "created_at": "2026-01-20T15:20:00Z",
        "updated_at": "2026-01-21T09:00:00Z",
        "days_since_created": 1
    }
]
```

### 8.2 Error Response

```json
{
    "title": [
        "Sarlavha kamida 3 ta belgidan iborat bo'lishi kerak!"
    ],
    "due_date": [
        "Tugash sanasi o'tmishda bo'lishi mumkin emas!"
    ]
}
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Blog Post API (Oson)

**Model:**
```python
class BlogPost(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.CharField(max_length=100)
    published = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
```

**Talablar:**
- ✅ Serializer yarating
- ✅ ViewSet bilan API qiling
- ✅ Admin panelda ro'yxatdan o'tkazing
- ✅ Custom action: `published` - faqat published postlar

### 📝 Topshiriq 2: Product API (O'rta)

**Model:**
```python
class Product(models.Model):
    CATEGORY_CHOICES = [
        ('electronics', 'Elektronika'),
        ('clothing', 'Kiyim'),
        ('food', 'Oziq-ovqat'),
    ]
    
    name = models.CharField(max_length=200)
    description = models.TextField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    category = models.CharField(max_length=20, choices=CATEGORY_CHOICES)
    in_stock = models.BooleanField(default=True)
    stock_quantity = models.IntegerField(default=0)
```

**Talablar:**
- ✅ Validation: price > 0
- ✅ Validation: stock_quantity >= 0
- ✅ Custom action: `low_stock` - stock < 10
- ✅ Custom field: `is_available` (in_stock AND stock > 0)

### 📝 Topshiriq 3: Note API with Categories (Qiyin)

**2 ta model:**
```python
class Category(models.Model):
    name = models.CharField(max_length=100)
    
class Note(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    category = models.ForeignKey(Category, on_delete=models.CASCADE)
    tags = models.CharField(max_length=200)  # Comma-separated
    pinned = models.BooleanField(default=False)
```

**Talablar:**
- ✅ Nested serializer (Note ichida Category ma'lumoti)
- ✅ Filtering by category
- ✅ Custom action: `pinned` - faqat pinned notes
- ✅ Validation: tags formatini tekshirish

---

## 🔗 KEYINGI DARSLAR

✅ **Dars 02 tugadi! API yaratishning barcha usullarini o'rgandingiz!**

**Keyingi darsda:**
- CRUD operatsiyalarini batafsil
- Generic views
- Mixins

---

## 📚 QISQA XULOSALAR

### FBV vs CBV vs ViewSet

| Xususiyat | FBV | APIView | ViewSet |
|-----------|-----|---------|---------|
| Oddiylik | ✅✅✅ | ✅✅ | ✅ |
| Flexibility | ✅✅ | ✅✅✅ | ✅ |
| Kod kamroq | ❌ | ✅ | ✅✅✅ |
| Router support | ❌ | ❌ | ✅✅✅ |
| **Tavsiya** | Oddiy API | Custom logic | Standard CRUD |

### Serializer Fields

```python
# Read-only
read_only_fields = ['id', 'created_at']

# Write-only (parol kabi)
extra_kwargs = {'password': {'write_only': True}}

# Custom method field
days_old = serializers.SerializerMethodField()
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**
