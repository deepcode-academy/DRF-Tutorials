# 🧩 1-DARS: DJANGO LOYIHASI VA DRF SOZLASH

## 🎯 Dars Maqsadi

Bu darsda siz Django va Django REST Framework (DRF) ni o'rnatish, yangi loyiha yaratish va birinchi ishlaydigan API ni sozlashni o'rganasiz.

**Dars oxirida siz:**
- ✅ Virtual environment yaratishni bilasiz
- ✅ Django va DRF ni to'g'ri o'rnatishni o'rganasiz
- ✅ Django loyihasi va ilova strukturasini tushunasiz
- ✅ Birinchi API endpoint yaratishni bilasiz
- ✅ DRF browsable API interfeysi bilan ishlashni o'rganasiz

---

## 📚 Boshlashdan Oldin

### Kerakli Bilimlar:
- Python asoslari
- Terminal/Command Line bilan ishlash
- Besic web development tushunchalari

### Kerakli Dasturlar:
- Python 3.8+ ([yuklab olish](https://www.python.org/downloads/))
- Code Editor (VS Code, PyCharm, Sublime Text)
- Terminal/Command Prompt

---

## 🚀 1. MUHITNI TAYYORLASH

### 1.1 Python Versiyasini Tekshirish

Birinchi navbatda Python o'rnatilganligini va versiyasini tekshiring:

```bash
python --version
# yoki
python3 --version
```

> **Eslatma:** Python 3.8 yoki undan yuqori versiya bo'lishi kerak.

### 1.2 Virtual Environment Yaratish

Virtual environment loyihangiz uchun alohida Python muhitini yaratadi. Bu boshqa loyihalarga ta'sir qilmaslikni ta'minlaydi.

**Virtual environment nima uchun kerak?**
- ✅ Kutubxonalar konfliktini oldini oladi
- ✅ Loyiha bog'liqliklarini alohida saqlaydi
- ✅ Loyihani boshqa kompyuterga ko'chirishni osonlashtiradi

```bash
# Virtual environment yaratish
python -m venv env

# Virtual environment ni faollashtirish
# Windows uchun:
env\Scripts\activate

# Linux/Mac uchun:
source env/bin/activate
```

**Faollashganini qanday bilaman?**
- Terminal oldida `(env)` ko'rinadi

**Virtual environment dan chiqish:**
```bash
deactivate
```

### 1.3 Django va DRF O'rnatish

Virtual environment faol holatda quyidagi buyruqni bajaring:

```bash
pip install django djangorestframework
```

**O'rnatilganini tekshirish:**
```bash
django-admin --version
```

---

## 🏗️ 2. DJANGO LOYIHASINI YARATISH

### 2.1 Yangi Loyiha Yaratish

```bash
django-admin startproject myproject .
```

> **Eslatma:** Oxiridagi nuqta `.` loyihani joriy papkada yaratadi.

### 2.2 Loyiha Strukturasi

Yaratilgan fayllar va ularning vazifalari:

```
myproject/
├── manage.py              # Loyihani boshqarish uchun asosiy fayl
├── myproject/
│   ├── __init__.py       # Python package marker
│   ├── settings.py       # Loyiha sozlamalari (Database, Apps, Middleware)
│   ├── urls.py           # Asosiy URL marshrutlari
│   ├── asgi.py           # ASGI deployment uchun
│   └── wsgi.py           # WSGI deployment uchun
```

**Muhim Fayllar:**
- **`manage.py`** - Loyihani ishga tushirish, migratsiya qilish va boshqa vazifalar uchun
- **`settings.py`** - Barcha konfiguratsiyalar
- **`urls.py`** - URL routing

### 2.3 Serverni Ishga Tushirish

```bash
python manage.py runserver
```

**Brauzerda tekshiring:**
- `http://127.0.0.1:8000/` manziliga o'ting
- "Welcome to Django" sahifasini ko'rishingiz kerak

**Server ishlayotganida:**
- `Ctrl + C` turishlari bilan to'xtating
- Kod o'zgartirilganda avtomatik qayta ishga tushadi

---

## 📦 3. DJANGO ILOVASINI YARATISH

### 3.1 Ilova Nima?

Django loyihalari bir nechta **ilovalardan** (apps) tashkil topadi. Har bir ilova alohida funksionallikni amalga oshiradi.

**Misol:**
- `users` - Foydalanuvchilar boshqaruvi
- `blog` - Blog postlari
- `api` - API endpoints

### 3.2 Ilova Yaratish

```bash
python manage.py startapp myapp
```

**Yaratilgan struktura:**

```
myapp/
├── __init__.py
├── admin.py          # Admin panel sozlamalari
├── apps.py           # Ilova konfiguratsiyasi
├── models.py         # Ma'lumotlar bazasi modellari
├── tests.py          # Test fayllar
├── views.py          # View funksiyalari
├── migrations/       # Database migratsiyalari
```

### 3.3 Ilovani Loyihaga Ro'yxatdan O'tkazish

`myproject/settings.py` faylini oching va `INSTALLED_APPS` ro'yxatini yangilang:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Bizning ilovalarimiz
    'myapp',
    
    # Third-party packages
    'rest_framework',
]
```

> **Muhim:** DRF ham `INSTALLED_APPS` ga qo'shilishi kerak!

---

## 💾 4. MA'LUMOTLAR BAZASI VA MODEL YARATISH

### 4.1 Model Nima?

Model - bu ma'lumotlar bazasidagi jadval strukturasini Python class orqali aniqlash.

**Afzalliklari:**
- SQL kod yozishga hojat yo'q
- Python orqali database bilan ishlash
- Database o'zgartirilsa, kod o'zgarmasligi mumkin

### 4.2 Oddiy Model Yaratish

`myapp/models.py` faylida quyidagi kodni yozing:

```python
from django.db import models

class Item(models.Model):
    """
    Oddiy Item modeli - mahsulot, kitob yoki boshqa narsalar uchun
    """
    name = models.CharField(max_length=100, verbose_name="Nom")
    description = models.TextField(verbose_name="Tavsif")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="Yaratilgan vaqt")
    updated_at = models.DateTimeField(auto_now=True, verbose_name="Yangilangan vaqt")

    class Meta:
        verbose_name = "Element"
        verbose_name_plural = "Elementlar"
        ordering = ['-created_at']  # Eng yangilari birinchi

    def __str__(self):
        return self.name
```

**Field turlari:**
- `CharField` - Qisqa matn (max_length majburiy)
- `TextField` - Uzun matn
- `DateTimeField` - Sana va vaqt
- `auto_now_add=True` - Faqat yaratilganda vaqt qo'yadi
- `auto_now=True` - Har safar yangilanganda vaqt yangilanadi

### 4.3 Migratsiyalarni Yaratish va Qo'llash

**1-qadam: Migratsiya fayllarini yaratish**

```bash
python manage.py makemigrations
```

Bu buyruq modeldan database uchun SQL buyruqlarni yaratadi.

**2-qadam: Migratsiyalarni qo'llash**

```bash
python manage.py migrate
```

Bu buyruq database da jadval yaratadi.

**Qanday ishlaydi?**
```
Model (Python) → makemigrations → Migration fayllar → migrate → Database jadvallari
```

---

## 🔧 5. DRF BILAN API YARATISH

### 5.1 Serializer Yaratish

Serializer - Model ma'lumotlarini JSON formatga aylantiradi va aksincha.

`myapp/serializers.py` faylini yarating:

```python
from rest_framework import serializers
from .models import Item

class ItemSerializer(serializers.ModelSerializer):
    """
    Item modelini JSON formatga aylantirish uchun serializer
    """
    class Meta:
        model = Item
        fields = ['id', 'name', 'description', 'created_at', 'updated_at']
        read_only_fields = ['id', 'created_at', 'updated_at']
```

**Izoh:**
- `fields` - JSON da ko'rsatiladigan maydonlar
- `read_only_fields` - Faqat o'qish uchun, o'zgartirib bo'lmaydi
- `fields = '__all__'` - Barcha maydonlarni ko'rsatish

### 5.2 ViewSet Yaratish

ViewSet - API uchun CRUD operatsiyalarini avtomatik ta'minlaydi.

`myapp/views.py` faylida:

```python
from rest_framework import viewsets
from rest_framework.permissions import AllowAny
from .models import Item
from .serializers import ItemSerializer

class ItemViewSet(viewsets.ModelViewSet):
    """
    Item CRUD API
    
    list: Barcha itemlarni olish
    create: Yangi item yaratish
    retrieve: Bitta itemni olish
    update: Itemni yangilash
    destroy: Itemni o'chirish
    """
    queryset = Item.objects.all()
    serializer_class = ItemSerializer
    permission_classes = [AllowAny]  # Hozircha hamma kirishi mumkin
```

**ViewSet afzalliklari:**
- CRUD avtomatik
- Kod kamroq
- Standard RESTful API

### 5.3 URL Routing (DefaultRouter)

`myproject/urls.py` faylini yangilang:

```python
from django.contrib import admin
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from myapp.views import ItemViewSet

# Router yaratish
router = DefaultRouter()
router.register(r'items', ItemViewSet, basename='item')

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include(router.urls)),  # API endpointlari
]
```

**Yaratilgan endpointlar:**

| HTTP Method | URL | Amal |
|-------------|-----|------|
| GET | `/api/items/` | Barcha itemlarni olish |
| POST | `/api/items/` | Yangi item yaratish |
| GET | `/api/items/1/` | 1-itemni olish |
| PUT | `/api/items/1/` | 1-itemni yangilash (to'liq) |
| PATCH | `/api/items/1/` | 1-itemni yangilash (qisman) |
| DELETE | `/api/items/1/` | 1-itemni o'chirish |

---

## ✅ 6. API NI SINAB KO'RISH

### 6.1 Serverni Ishga Tushirish

```bash
python manage.py runserver
```

### 6.2 DRF Browsable API

Brauzerda quyidagi manzillarga o'ting:

**1. Barcha itemlar:**
- `http://127.0.0.1:8000/api/items/`

**DRF interfeysi orqali:**
- ✅ GET - Itemlarni ko'rish
- ✅ POST - Yangi item qo'shish (pastdagi forma orqali)

**2. Bitta item:**
- `http://127.0.0.1:8000/api/items/1/`

### 6.3 POST Bilan Yangi Item Yaratish

DRF interfeysi pastida forma bor:

```json
{
    "name": "Birinchi mahsulot",
    "description": "Bu test mahsuloti"
}
```

**POST** tugmasini bosing.

### 6.4 cURL orqali Test Qilish

**GET so'rovi:**
```bash
curl http://127.0.0.1:8000/api/items/
```

**POST so'rovi:**
```bash
curl -X POST http://127.0.0.1:8000/api/items/ \
  -H "Content-Type: application/json" \
  -d '{"name":"Test Item","description":"Test description"}'
```

---

## 🛡️ 7. ADMIN PANEL SOZLASH

### 7.1 Superuser Yaratish

```bash
python manage.py createsuperuser
```

**So'raladi:**
- Username
- Email
- Password

### 7.2 Modelni Admin Panelda Ro'yxatdan O'tkazish

`myapp/admin.py` faylida:

```python
from django.contrib import admin
from .models import Item

@admin.register(Item)
class ItemAdmin(admin.ModelAdmin):
    """
    Item modelini admin panelda ko'rsatish uchun
    """
    list_display = ['id', 'name', 'created_at', 'updated_at']
    list_filter = ['created_at']
    search_fields = ['name', 'description']
    readonly_fields = ['created_at', 'updated_at']
```

**Imkoniyatlar:**
- `list_display` - Jadvalda ko'rsatiladigan ustunlar
- `list_filter` - Filtr qo'shish
- `search_fields` - Qidiruv
- `readonly_fields` - O'zgartirib bo'lmaydigan maydonlar

### 7.3 Admin Paneliga Kirish

```
http://127.0.0.1:8000/admin/
```

Yaratgan username va parolingiz bilan kiring.

---

## 📊 8. LOYIHA STRUKTURASI (YAKUNIY)

```
myproject/
├── manage.py
├── myproject/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
├── myapp/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py          # Item modeli
│   ├── serializers.py     # ItemSerializer
│   ├── views.py           # ItemViewSet
│   ├── tests.py
│   └── migrations/
└── env/                   # Virtual environment
```

---

## 💡 9. MUHIM ESLATMALAR

### ✅ Virtual Environment

- **Har doim** virtual environment ichida ishlang
- Yangi terminal ochganingizda `env\Scripts\activate` ni unutmang

### ✅ Migratsiyalar

- Model o'zgartirganda **albatta** migratsiya bajaring:
  ```bash
  python manage.py makemigrations
  python manage.py migrate
  ```

### ✅ Git Ignore

`.gitignore` faylida quyidagilarni qo'shing:

```
env/
*.pyc
__pycache__/
db.sqlite3
.DS_Store
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Book API (Oson)

Book modeli yaratib, API qiling:

**Model:**
- title (CharField)
- author (CharField)
- published_year (IntegerField)
- isbn (CharField)

**Talablar:**
- Serializer yarating
- ViewSet yarating
- URL routing qo'shing
- Admin panelda ro'yxatdan o'tkazing

### 📝 Topshiriq 2: Student API (O'rta)

Student modeli yaratib, ko'proq field qo'shing:

**Model:**
- first_name
- last_name
- email (EmailField, unique=True)
- phone_number
- date_of_birth (DateField)
- is_active (BooleanField)

**Qo'shimcha:**
- `__str__` metodida full_name qaytaring
- Admin panelda search va filter qo'shing

### 📝 Topshiriq 3: Product API (Qiyin)

Product modeli bilan ishlang:

**Model:**
- name
- description
- price (DecimalField)
- stock (IntegerField)
- category (CharField with choices)
- is_available (BooleanField)

**Qo'shimcha:**
- Serializer da faqat mavjud mahsulotlarni ko'rsating
- ViewSet da custom querysetlar yarating

---

## 🔗 KEYINGI DARSLAR

✅ **Dars 01 tugadi! Siz endi Django va DRF bilan ishlashni boshlang!**

**Keyingi darsda:**
- Birinchi API ni batafsil yaratamiz
- Serializer validation
- Custom API view'lar

---

## 📚 QULAYLIK UCHUN BUYRUQLAR RO'YXATI

```bash
# Virtual environment
python -m venv env
env\Scripts\activate          # Windows
source env/bin/activate       # Linux/Mac

# O'rnatish
pip install django djangorestframework

# Loyiha yaratish
django-admin startproject myproject .
python manage.py startapp myapp

# Migratsiyalar
python manage.py makemigrations
python manage.py migrate

# Server
python manage.py runserver

# Admin
python manage.py createsuperuser
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**
