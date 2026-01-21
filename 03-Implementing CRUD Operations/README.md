# 🧩 3-DARS: CRUD OPERATSIYALARINI AMALGA OSHIRISH

## 🎯 Dars Maqsadi

Bu darsda CRUD (Create, Read, Update, Delete) operatsiyalarini to'liq o'rganasiz. **Telefon Kitobchasi (Contact Book)** loyihasini yaratish orqali har bir CRUD operatsiyasini amaliyotda ko'rasiz.

**Dars oxirida siz:**
- ✅ CREATE - Yangi kontakt qo'shishni bilasiz
- ✅ READ - Kontaktlarni ko'rishni o'rganasiz (hamma va bitta)
- ✅ UPDATE - Kontakt ma'lumotlarini yangilashni bilasiz
- ✅ DELETE - Kontaktni o'chirishni o'rganasiz
- ✅ Validation va Error handling qilishni bilasiz
- ✅ PATCH vs PUT farqini tushunasiz

---

## 💡 Nima Uchun Contact Book?

**Telefon kitobchasi** - eng oddiy va tushunarli misol:
- 📱 Har kim biladi telefon kitobchasi qanday ishlashini
- ✅ CRUD operatsiyalari aniq ko'rinadi
- 🔍 Qidirish va filtrlash oson
- 📊 Real-world example

---

## 🚀 1. YANGI LOYIHA YARATISH

### 1.1 Loyiha va Ilova Yaratish

**Yangi papka yarating va virtual environment ishga tushiring:**

```bash
# Yangi papka yaratish
mkdir contact_book_api
cd contact_book_api

# Virtual environment
python -m venv env

# Activate (Windows)
env\Scripts\activate

# Django va DRF o'rnatish
pip install django djangorestframework
```

**Django loyihasi yaratish:**

```bash
# Loyiha yaratish
django-admin startproject contact_project .

# Ilova yaratish
python manage.py startapp contacts
```

### 1.2 Settings.py Sozlash

`contact_project/settings.py` faylini oching:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Third-party apps
    'rest_framework',  # DRF ni qo'shamiz
    
    # Local apps
    'contacts',  # Bizning ilova
]

# DRF sozlamalari (ixtiyoriy)
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10  # Bir sahifada 10 ta kontakt
}
```

---

## 💾 2. MODEL YARATISH

### 2.1 Contact Modeli

`contacts/models.py` faylida:

```python
from django.db import models

class Contact(models.Model):
    """
    Telefon kitobchasidagi kontakt modeli
    Bu model har bir kontaktning ma'lumotlarini saqlaydi
    """
    
    # Kontaktning ismi (majburiy)
    # CharField - qisqa matn uchun ishlatiladi
    # max_length=100 - maksimal 100 ta belgi
    first_name = models.CharField(
        max_length=100,
        verbose_name="Ism",
        help_text="Kontaktning ismi"
    )
    
    # Kontaktning familiyasi (majburiy)
    last_name = models.CharField(
        max_length=100,
        verbose_name="Familiya",
        help_text="Kontaktning familiyasi"
    )
    
    # Telefon raqami (majburiy va unique - takrorlanmasligi kerak)
    # unique=True - bir xil telefon raqam 2 marta bo'lmasligi kerak
    phone_number = models.CharField(
        max_length=20,
        unique=True,
        verbose_name="Telefon raqami",
        help_text="+998901234567 formatida"
    )
    
    # Elektron pochta (ixtiyoriy)
    # blank=True - forma to'ldirilganda majburiy emas
    # null=True - database da NULL qiymat bo'lishi mumkin
    email = models.EmailField(
        blank=True,
        null=True,
        verbose_name="Email",
        help_text="Elektron pochta manzili"
    )
    
    # Manzil (ixtiyoriy)
    # TextField - uzun matn uchun ishlatiladi
    address = models.TextField(
        blank=True,
        null=True,
        verbose_name="Manzil",
        help_text="To'liq manzil"
    )
    
    # Tug'ilgan kun (ixtiyoriy)
    # DateField - faqat sana (vaqtsiz)
    date_of_birth = models.DateField(
        blank=True,
        null=True,
        verbose_name="Tug'ilgan kun"
    )
    
    # Sevimli kontaktmi? (checkbox)
    # BooleanField - True/False qiymat
    # default=False - standart qiymati False (sevimli emas)
    is_favorite = models.BooleanField(
        default=False,
        verbose_name="Sevimli",
        help_text="Sevimli kontaktlar ro'yxatiga qo'shish"
    )
    
    # Yaratilgan vaqt (avtomatik)
    # auto_now_add=True - faqat birinchi marta yaratilganda vaqt qo'yiladi
    created_at = models.DateTimeField(
        auto_now_add=True,
        verbose_name="Yaratilgan vaqt"
    )
    
    # Yangilangan vaqt (avtomatik)
    # auto_now=True - har safar yangilanganda vaqt yangilanadi
    updated_at = models.DateTimeField(
        auto_now=True,
        verbose_name="Yangilangan vaqt"
    )
    
    class Meta:
        # Admin panelda ko'rinish nomlari
        verbose_name = "Kontakt"
        verbose_name_plural = "Kontaktlar"
        
        # Standart tartiblash - ismga qarab A-Z
        ordering = ['first_name', 'last_name']
        
        # Database indekslari - qidirishni tezlashtirish uchun
        indexes = [
            models.Index(fields=['phone_number']),
            models.Index(fields=['email']),
        ]
    
    def __str__(self):
        """
        Admin panelda va shell da obyekt nomini ko'rsatish
        Masalan: "Alisher Navoiy"
        """
        return f"{self.first_name} {self.last_name}"
    
    def get_full_name(self):
        """
        To'liq ismni olish uchun custom metod
        """
        return f"{self.first_name} {self.last_name}"
    
    def get_age(self):
        """
        Yoshni hisoblash (agar tug'ilgan kun kiritilgan bo'lsa)
        """
        if self.date_of_birth:
            from datetime import date
            today = date.today()
            age = today.year - self.date_of_birth.year
            # Tug'ilgan kun hali bo'lmagan bo'lsa, 1 yil kamaytiramiz
            if today.month < self.date_of_birth.month or \
               (today.month == self.date_of_birth.month and today.day < self.date_of_birth.day):
                age -= 1
            return age
        return None
```

### 2.2 Migratsiya Qilish

```bash
# Migratsiya fayllarini yaratish
python manage.py makemigrations

# Natija:
# Migrations for 'contacts':
#   contacts/migrations/0001_initial.py
#     - Create model Contact

# Database ga tatbiq qilish
python manage.py migrate
```

---

## 🔧 3. SERIALIZER YARATISH

### 3.1 Serializer Nima?

**Serializer** - ma'lumotlarni JSON formatga va qaytadan Python obyektga aylantiruvchi vosita.

```
Contact Model (Python) → Serializer → JSON (frontend uchun)
JSON (frontend dan) → Serializer → Contact Model (Python)
```

### 3.2 ContactSerializer

`contacts/serializers.py` faylini yarating:

```python
from rest_framework import serializers
from .models import Contact

class ContactSerializer(serializers.ModelSerializer):
    """
    Contact modelini JSON formatga aylantirish uchun serializer
    """
    
    # Read-only field - faqat response da ko'rinadi, create/update da yo'q
    full_name = serializers.SerializerMethodField()
    age = serializers.SerializerMethodField()
    
    class Meta:
        model = Contact  # Qaysi model ishlatiladi
        
        # JSON da qaysi fieldlar ko'rinadi
        fields = [
            'id',              # Avtomatik yaratilgan ID
            'first_name',      # Ism
            'last_name',       # Familiya
            'full_name',       # To'liq ism (custom field)
            'phone_number',    # Telefon
            'email',           # Email
            'address',         # Manzil
            'date_of_birth',   # Tug'ilgan kun
            'age',             # Yosh (custom field)
            'is_favorite',     # Sevimli
            'created_at',      # Yaratilgan vaqt
            'updated_at',      # Yangilangan vaqt
        ]
        
        # Faqat o'qish uchun fieldlar (POST/PUT da o'zgartirib bo'lmaydi)
        read_only_fields = ['id', 'created_at', 'updated_at']
    
    def get_full_name(self, obj):
        """
        To'liq ismni qaytarish
        obj - Contact obyekti
        """
        return obj.get_full_name()
    
    def get_age(self, obj):
        """
        Yoshni qaytarish
        """
        return obj.get_age()
    
    # VALIDATION - Ma'lumotlarni tekshirish
    
    def validate_phone_number(self, value):
        """
        Telefon raqamini tekshirish
        value - foydalanuvchi yuborgan telefon raqam
        """
        # Bo'sh joylarni olib tashlash
        value = value.strip()
        
        # Kamida 9 ta raqam bo'lishi kerak
        if len(value) < 9:
            raise serializers.ValidationError(
                "Telefon raqam kamida 9 ta raqamdan iborat bo'lishi kerak!"
            )
        
        # Faqat raqam, +, -, (, ) bo'lishi mumkin
        allowed_chars = set('0123456789+-() ')
        if not all(char in allowed_chars for char in value):
            raise serializers.ValidationError(
                "Telefon raqamda faqat raqamlar va +, -, (, ) belgilari bo'lishi mumkin!"
            )
        
        return value
    
    def validate_email(self, value):
        """
        Email validatsiyasi
        EmailField avtomatik tekshiradi, lekin qo'shimcha tekshirishlar qo'shamiz
        """
        if value:  # Agar email berilgan bo'lsa
            # Kichik harflarga aylantirish
            value = value.lower().strip()
        return value
    
    def validate_first_name(self, value):
        """
        Ism validatsiyasi
        """
        # Bo'sh joylarni olib tashlash
        value = value.strip()
        
        # Kamida 2 ta belgi bo'lishi kerak
        if len(value) < 2:
            raise serializers.ValidationError(
                "Ism kamida 2 ta belgidan iborat bo'lishi kerak!"
            )
        
        # Birinchi harfni katta qilish
        value = value.capitalize()
        
        return value
    
    def validate_last_name(self, value):
        """
        Familiya validatsiyasi
        """
        value = value.strip()
        
        if len(value) < 2:
            raise serializers.ValidationError(
                "Familiya kamida 2 ta belgidan iborat bo'lishi kerak!"
            )
        
        value = value.capitalize()
        
        return value
    
    def validate(self, data):
        """
        Barcha fieldlarni birga tekshirish
        data - barcha yuborilgan ma'lumotlar dictionary ko'rinishida
        """
        # Agar tug'ilgan kun kiritilgan bo'lsa, kelajakda bo'lmasligi kerak
        if 'date_of_birth' in data and data['date_of_birth']:
            from datetime import date
            if data['date_of_birth'] > date.today():
                raise serializers.ValidationError({
                    'date_of_birth': "Tug'ilgan kun kelajakda bo'lishi mumkin emas!"
                })
        
        return data
```

### 3.3 Qisqa Serializer (List uchun)

```python
class ContactListSerializer(serializers.ModelSerializer):
    """
    Ro'yxat ko'rinishi uchun qisqaroq serializer
    Faqat asosiy ma'lumotlar
    """
    full_name = serializers.SerializerMethodField()
    
    class Meta:
        model = Contact
        fields = ['id', 'full_name', 'phone_number', 'is_favorite']
    
    def get_full_name(self, obj):
        return obj.get_full_name()
```

---

## 🎨 4. CRUD VIEWS YARATISH

### 4.1 Function-Based Views

`contacts/views.py` faylida:

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from .models import Contact
from .serializers import ContactSerializer, ContactListSerializer

# ============================================
# CREATE va READ (LIST) - Barcha kontaktlar
# ============================================

@api_view(['GET', 'POST'])
def contact_list(request):
    """
    GET:  Barcha kontaktlarni olish
    POST: Yangi kontakt yaratish
    
    URL: /api/contacts/
    """
    
    # -------- GET - Barcha kontaktlarni olish --------
    if request.method == 'GET':
        # Database dan barcha kontaktlarni olish
        contacts = Contact.objects.all()
        
        # Qisqa serializer ishlatamiz (list uchun)
        serializer = ContactListSerializer(contacts, many=True)
        # many=True degani - bir nechta obyektni serialize qilish
        
        # JSON formatda qaytarish
        return Response(serializer.data)
    
    # -------- POST - Yangi kontakt yaratish --------
    elif request.method == 'POST':
        # Foydalanuvchi yuborgan ma'lumotlarni olish
        # request.data - JSON formatdagi ma'lumotlar
        serializer = ContactSerializer(data=request.data)
        
        # Ma'lumotlar to'g'rimi? (validation)
        if serializer.is_valid():
            # Agar to'g'ri bo'lsa, database ga saqlash
            serializer.save()
            
            # 201 Created status code bilan qaytarish
            return Response(
                serializer.data,
                status=status.HTTP_201_CREATED
            )
        
        # Agar validation xatosi bo'lsa
        # 400 Bad Request status code bilan xatolarni qaytarish
        return Response(
            serializer.errors,
            status=status.HTTP_400_BAD_REQUEST
        )


# ============================================
# READ (DETAIL), UPDATE, DELETE - Bitta kontakt
# ============================================

@api_view(['GET', 'PUT', 'PATCH', 'DELETE'])
def contact_detail(request, pk):
    """
    GET:    Bitta kontaktni olish
    PUT:    Kontaktni to'liq yangilash
    PATCH:  Kontaktni qisman yangilash
    DELETE: Kontaktni o'chirish
    
    URL: /api/contacts/<id>/
    pk: Primary Key (kontakt ID si)
    """
    
    # -------- Kontaktni topish --------
    try:
        # pk (primary key) orqali database dan qidirish
        contact = Contact.objects.get(pk=pk)
    except Contact.DoesNotExist:
        # Agar topilmasa, 404 Not Found qaytarish
        return Response(
            {'error': f'ID={pk} kontakt topilmadi!'},
            status=status.HTTP_404_NOT_FOUND
        )
    
    # -------- GET - Kontaktni ko'rish --------
    if request.method == 'GET':
        # Kontaktni serialize qilish
        serializer = ContactSerializer(contact)
        
        # JSON formatda qaytarish
        return Response(serializer.data)
    
    # -------- PUT - To'liq yangilash --------
    elif request.method == 'PUT':
        # PUT - BARCHA fieldlarni yuborish kerak
        # Masalan: first_name, last_name, phone_number - HAMMASI
        
        serializer = ContactSerializer(
            contact,                # Qaysi kontaktni yangilash
            data=request.data       # Yangi ma'lumotlar
        )
        
        if serializer.is_valid():
            serializer.save()  # Database ga saqlash
            return Response(serializer.data)
        
        return Response(
            serializer.errors,
            status=status.HTTP_400_BAD_REQUEST
        )
    
    # -------- PATCH - Qisman yangilash --------
    elif request.method == 'PATCH':
        # PATCH - Faqat o'zgartirmoqchi bo'lgan fieldlarni yuborish
        # Masalan: faqat phone_number ni o'zgartirish
        
        serializer = ContactSerializer(
            contact,
            data=request.data,
            partial=True  # partial=True - qisman yangilash ruxsati
        )
        
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        
        return Response(
            serializer.errors,
            status=status.HTTP_400_BAD_REQUEST
        )
    
    # -------- DELETE - O'chirish --------
    elif request.method == 'DELETE':
        # Database dan o'chirish
        contact.delete()
        
        # 204 No Content - muvaffaqiyatli o'chirildi, content yo'q
        return Response(
            {'message': 'Kontakt muvaffaqiyatli o'chirildi!'},
            status=status.HTTP_204_NO_CONTENT
        )


# ============================================
# QIDIRISH - Ism bo'yicha qidirish
# ============================================

@api_view(['GET'])
def search_contacts(request):
    """
    Ismga qarab kontaktlarni qidirish
    
    URL: /api/contacts/search/?q=Ali
    """
    
    # URL dan 'q' parametrini olish
    # Masalan: /api/contacts/search/?q=Ali
    query = request.GET.get('q', '')  # Default: bo'sh string
    
    if not query:
        return Response(
            {'error': 'Qidiruv so\'zi kiritilmagan!'},
            status=status.HTTP_400_BAD_REQUEST
        )
    
    # Ism yoki familiyada qidiruv so'zi bor kontaktlarni topish
    # icontains - katta-kichik harf farqsiz qidirish
    contacts = Contact.objects.filter(
        first_name__icontains=query
    ) | Contact.objects.filter(
        last_name__icontains=query
    )
    
    serializer = ContactListSerializer(contacts, many=True)
    
    return Response({
        'query': query,
        'count': contacts.count(),
        'results': serializer.data
    })


# ============================================
# SEVIMLILAR - Sevimli kontaktlar
# ============================================

@api_view(['GET'])
def favorite_contacts(request):
    """
    Faqat sevimli kontaktlarni olish
    
    URL: /api/contacts/favorites/
    """
    
    # is_favorite=True bo'lgan kontaktlarni filterlash
    favorites = Contact.objects.filter(is_favorite=True)
    
    serializer = ContactListSerializer(favorites, many=True)
    
    return Response({
        'count': favorites.count(),
        'favorites': serializer.data
    })
```

---

## 🌐 5. URL ROUTING

`contacts/urls.py` faylini yarating:

```python
from django.urls import path
from . import views

# app_name - URL reverse uchun
app_name = 'contacts'

urlpatterns = [
    # CREATE va LIST - Barcha kontaktlar
    # GET:  Barcha kontaktlarni ko'rish
    # POST: Yangi kontakt yaratish
    path(
        'contacts/',
        views.contact_list,
        name='contact-list'
    ),
    
    # DETAIL - Bitta kontakt
    # GET:    Kontaktni ko'rish
    # PUT:    To'liq yangilash
    # PATCH:  Qisman yangilash
    # DELETE: O'chirish
    path(
        'contacts/<int:pk>/',
        views.contact_detail,
        name='contact-detail'
    ),
    
    # SEARCH - Qidirish
    path(
        'contacts/search/',
        views.search_contacts,
        name='contact-search'
    ),
    
    # FAVORITES - Sevimlilar
    path(
        'contacts/favorites/',
        views.favorite_contacts,
        name='contact-favorites'
    ),
]
```

`contact_project/urls.py` asosiy fayliga qo'shish:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('contacts.urls')),  # API URLs
]
```

---

## 🛡️ 6. ADMIN PANEL

`contacts/admin.py`:

```python
from django.contrib import admin
from .models import Contact

@admin.register(Contact)
class ContactAdmin(admin.ModelAdmin):
    """
    Contact modelini admin panelda boshqarish
    """
    
    # Ro'yxatda ko'rinadigan ustunlar
    list_display = [
        'id',
        'first_name',
        'last_name',
        'phone_number',
        'email',
        'is_favorite',
        'created_at'
    ]
    
    # Ro'yxatda filterlash
    list_filter = ['is_favorite', 'created_at']
    
    # Qidiruv maydonlari
    search_fields = [
        'first_name',
        'last_name',
        'phone_number',
        'email'
    ]
    
    # Ro'yxatda o'zgartirishga ruxsat
    list_editable = ['is_favorite']
    
    # Faqat o'qish uchun
    readonly_fields = ['created_at', 'updated_at']
    
    # Sana bo'yicha navigatsiya
    date_hierarchy = 'created_at'
    
    # Har bir sahifada nechta element
    list_per_page = 20
```

**Superuser yaratish:**

```bash
python manage.py createsuperuser
```

---

## ✅ 7. API TESTING

### 7.1 Server Ishga Tushirish

```bash
python manage.py runserver
```

### 7.2 DRF Browsable API

**URL'lar:**
- `http://127.0.0.1:8000/api/contacts/` - Barcha kontaktlar
- `http://127.0.0.1:8000/api/contacts/1/` - 1-kontakt
- `http://127.0.0.1:8000/api/contacts/search/?q=Ali` - Qidirish
- `http://127.0.0.1:8000/api/contacts/favorites/` - Sevimlilar

### 7.3 CRUD Operatsiyalari - cURL

**CREATE - Yangi kontakt:**

```bash
curl -X POST http://127.0.0.1:8000/api/contacts/ \
  -H "Content-Type: application/json" \
  -d '{
    "first_name": "Alisher",
    "last_name": "Navoiy",
    "phone_number": "+998901234567",
    "email": "alisher@example.com",
    "address": "Toshkent, Chilonzor"
  }'
```

**READ - Barcha kontaktlar:**

```bash
curl http://127.0.0.1:8000/api/contacts/
```

**READ - Bitta kontakt:**

```bash
curl http://127.0.0.1:8000/api/contacts/1/
```

**UPDATE (PUT) - To'liq yangilash:**

```bash
curl -X PUT http://127.0.0.1:8000/api/contacts/1/ \
  -H "Content-Type: application/json" \
  -d '{
    "first_name": "Alisher",
    "last_name": "Navoiy",
    "phone_number": "+998901234567",
    "email": "alisher.navoiy@example.com",
    "address": "Toshkent, Yunusobod"
  }'
```

**UPDATE (PATCH) - Qisman yangilash:**

```bash
curl -X PATCH http://127.0.0.1:8000/api/contacts/1/ \
  -H "Content-Type: application/json" \
  -d '{"phone_number": "+998909999999"}'
```

**DELETE - O'chirish:**

```bash
curl -X DELETE http://127.0.0.1:8000/api/contacts/1/
```

---

## 📊 8. PUT vs PATCH FARQI

### PUT - To'liq Yangilash

```json
{
  "first_name": "Alisher",
  "last_name": "Navoiy",
  "phone_number": "+998901234567",
  "email": "alisher@example.com",
  "address": "Toshkent"
}
```

**BARCHA fieldlarni yuborish kerak!** Agar birorta field yo'q bo'lsa, o'sha field tozalanadi.

### PATCH - Qisman Yangilash

```json
{
  "phone_number": "+998909999999"
}
```

**Faqat o'zgartirmoqchi bo'lgan fieldlarni yuborish!** Boshqa fieldlar o'zgarishsiz qoladi.

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Validation Qo'shish (Oson)

**Maqsad:** Qo'shimcha validatsiyalar qo'shish

1. Email domainini tekshirish (faqat gmail.com, mail.ru)
2. Telefon raqam +998 bilan boshlanishi kerak
3. Ism faqat harflardan iborat bo'lishi kerak

### 📝 Topshiriq 2: Custom Actions (O'rta)

**Maqsad:** Qo'shimcha endpoint'lar yaratish

1. `/api/contacts/toggle-favorite/<id>/` - Sevimli/oddiy o'zgartirish
2. `/api/contacts/birthday-today/` - Bugun tu'gilgan kunlari
3. `/api/contacts/count/` - Jami kontaktlar soni

### 📝 Topshiriq 3: Bulk Operations (Qiyin)

**Maqsad:** Ko'p kontaktlar bilan ishlash

1. Bir nechta kontaktni bir vaqtda yaratish (bulk create)
2. Bir nechta kontaktni bir vaqtda o'chirish (bulk delete)
3. Export - Barcha kontaktlarni CSV formatda yuklab olish

---

## 🔗 KEYINGI DARSLAR

✅ **Dars 03 tugadi! CRUD operatsiyalarini to'liq o'rgandingiz!**

**Keyingi darsda:**
- ModelSerializer batafsil
- Nested serializers
- Custom fields

---

## 📚 QISQA XULOSA

### CRUD Operatsiyalari

| Operatsiya | HTTP Method | URL | Vazifasi |
|-----------|-------------|-----|----------|
| **C**reate | POST | `/api/contacts/` | Yangi kontakt yaratish |
| **R**ead (List) | GET | `/api/contacts/` | Barcha kontaktlar |
| **R**ead (Detail) | GET | `/api/contacts/1/` | Bitta kontakt |
| **U**pdate (To'liq) | PUT | `/api/contacts/1/` | To'liq yangilash |
| **U**pdate (Qisman) | PATCH | `/api/contacts/1/` | Qisman yangilash |
| **D**elete | DELETE | `/api/contacts/1/` | O'chirish |

### HTTP Status Codes

| Code | Ma'nosi | Qachon? |
|------|---------|---------|
| 200 | OK | Muvaffaqiyatli (GET, PUT, PATCH) |
| 201 | Created | Yangi yaratildi (POST) |
| 204 | No Content | O'chirildi (DELETE) |
| 400 | Bad Request | Validation xatosi |
| 404 | Not Found | Topilmadi |
| 500 | Server Error | Server xatosi |

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**
