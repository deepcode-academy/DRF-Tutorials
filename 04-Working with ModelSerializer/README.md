# 🧩 4-DARS: MODELSERIALIZER BILAN ISHLASH

## 🎯 Dars Maqsadi

Bu darsda ModelSerializer ning barcha imkoniyatlari bilan ishlashni o'rganasiz. **Kutubxona Boshqaruv Tizimi (Library Management System)** loyihasini yaratish orqali nested serializers, custom fields, validation va boshqa ilg'or texnikalarni amaliyotda ko'rasiz.

**Dars oxirida siz:**
- ✅ ModelSerializer ning barcha xususiyatlarini bilasiz
- ✅ Nested Serializers (bog'langan modellar) yaratishni o'rganasiz
- ✅ Custom Fields va SerializerMethodField ishlatishni bilasiz
- ✅ Field-level va Object-level validation qilishni o'rganasiz
- ✅ read_only, write_only fieldlar bilan ishlashni bilasiz
- ✅ extra_kwargs orqali konfiguratsiya qilishni o'rganasiz

---

## 💡 Nima Uchun Library System?

**Kutubxona tizimi** - ModelSerializer uchun eng yaxshi misol:
- 📚 2 ta bog'langan model: Author va Book
- 🔗 Nested relationships ko'rsatish oson
- ✅ Validation qoidalari aniq
- 📊 Real-world application

---

## 🚀 1. YANGI LOYIHA YARATISH

### 1.1 Loyiha Sozlash

```bash
# Yangi papka
mkdir library_api
cd library_api

# Virtual environment
python -m venv env
env\Scripts\activate  # Windows

# O'rnatish
pip install django djangorestframework

# Loyiha yaratish
django-admin startproject library_project .

# Ilova yaratish
python manage.py startapp books
```

### 1.2 Settings Sozlash

`library_project/settings.py`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Third-party
    'rest_framework',
    
    # Local apps
    'books',
]
```

---

## 💾 2. MODELS YARATISH

### 2.1 Author va Book Modellari

`books/models.py`:

```python
from django.db import models
from django.core.validators import MinValueValidator, MaxValueValidator

class Author(models.Model):
    """
    Muallif (Yozuvchi) modeli
    Har bir muallif bir nechta kitob yozishi mumkin
    """
    
    # Muallif ismi (majburiy)
    # CharField - qisqa matn (maksimal uzunlik bilan)
    first_name = models.CharField(
        max_length=100,
        verbose_name="Ism",
        help_text="Muallifning ismi"
    )
    
    # Muallif familiyasi (majburiy)
    last_name = models.CharField(
        max_length=100,
        verbose_name="Familiya"
    )
    
    # Tug'ilgan yil (ixtiyoriy)
    # IntegerField - butun son
    # validators - qo'shimcha tekshiruvlar
    birth_year = models.IntegerField(
        blank=True,
        null=True,
        verbose_name="Tug'ilgan yil",
        validators=[
            MinValueValidator(1800),  # Minimal 1800-yil
            MaxValueValidator(2024)   # Maksimal 2024-yil
        ]
    )
    
    # Biografiya (ixtiyoriy)
    # TextField - uzun matn (uzunlik chegarasiz)
    biography = models.TextField(
        blank=True,
        null=True,
        verbose_name="Biografiya",
        help_text="Muallif haqida qisqacha ma'lumot"
    )
    
    # Mamlakat (ixtiyoriy)
    country = models.CharField(
        max_length=100,
        blank=True,
        null=True,
        verbose_name="Mamlakat"
    )
    
    # Avtomatik vaqt maydonlari
    created_at = models.DateTimeField(
        auto_now_add=True,
        verbose_name="Qo'shilgan vaqt"
    )
    
    class Meta:
        verbose_name = "Muallif"
        verbose_name_plural = "Mualliflar"
        # Familiya, keyin ism bo'yicha tartiblash
        ordering = ['last_name', 'first_name']
    
    def __str__(self):
        """
        Admin panelda va API da ko'rinish
        Return: "Alisher Navoiy"
        """
        return f"{self.first_name} {self.last_name}"
    
    def get_full_name(self):
        """
        To'liq ismni olish uchun custom metod
        """
        return f"{self.first_name} {self.last_name}"
    
    def get_age(self):
        """
        Yoshi (agar tug'ilgan yil berilgan bo'lsa)
        """
        if self.birth_year:
            from datetime import datetime
            current_year = datetime.now().year
            return current_year - self.birth_year
        return None


class Book(models.Model):
    """
    Kitob modeli
    Har bir kitob bitta muallifga tegishli
    """
    
    # Kitob sarlavhasi (majburiy)
    title = models.CharField(
        max_length=300,
        verbose_name="Sarlavha",
        help_text="Kitob nomi"
    )
    
    # Muallif (Foreign Key - bog'lanish)
    # ForeignKey - boshqa modelga bog'lanish
    # on_delete=models.CASCADE - muallif o'chirilsa, uning kitoblari ham o'chiriladi
    # related_name='books' - Author obyektidan kitoblarni olish: author.books.all()
    author = models.ForeignKey(
        Author,
        on_delete=models.CASCADE,
        related_name='books',
        verbose_name="Muallif",
        help_text="Kitob muallifi"
    )
    
    # ISBN (International Standard Book Number) - unikal kitob kodi
    # unique=True - bir xil ISBN 2 marta bo'lmasligi kerak
    isbn = models.CharField(
        max_length=13,
        unique=True,
        verbose_name="ISBN",
        help_text="13 raqamli ISBN kod"
    )
    
    # Chop etilgan yil (majburiy)
    published_year = models.IntegerField(
        verbose_name="Chop etilgan yil",
        validators=[
            MinValueValidator(1000),
            MaxValueValidator(2024)
        ]
    )
    
    # Sahifalar soni (ixtiyoriy)
    pages = models.IntegerField(
        blank=True,
        null=True,
        verbose_name="Sahifalar soni",
        validators=[MinValueValidator(1)]
    )
    
    # Janr/Tur (tanlov)
    # choices - faqat ro'yxatdagi qiymatlardan birini tanlash
    GENRE_CHOICES = [
        ('fiction', 'Badiiy adabiyot'),
        ('non_fiction', 'Ilmiy adabiyot'),
        ('poetry', 'She\'riyat'),
        ('drama', 'Drama'),
        ('science', 'Fan'),
        ('history', 'Tarix'),
        ('biography', 'Biografiya'),
        ('other', 'Boshqa'),
    ]
    
    genre = models.CharField(
        max_length=20,
        choices=GENRE_CHOICES,
        default='fiction',
        verbose_name="Janr"
    )
    
    # Tavsif (ixtiyoriy)
    description = models.TextField(
        blank=True,
        null=True,
        verbose_name="Tavsif",
        help_text="Kitob haqida qisqacha"
    )
    
    # Nusxalar soni kutubxonada (kutubxonada nechta kitob bor)
    copies_available = models.IntegerField(
        default=1,
        verbose_name="Mavjud nusxalar",
        validators=[MinValueValidator(0)]
    )
    
    # Ijara berilganmi? (boolean)
    is_available = models.BooleanField(
        default=True,
        verbose_name="Mavjud",
        help_text="Kutubxonada bormi?"
    )
    
    # Avtomatik vaqt maydonlari
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name = "Kitob"
        verbose_name_plural = "Kitoblar"
        # Muallif, keyin sarlavha bo'yicha
        ordering = ['author', 'title']
        # Database indeksi - qidirishni tezlashtirish
        indexes = [
            models.Index(fields=['isbn']),
            models.Index(fields=['author']),
        ]
    
    def __str__(self):
        """
        Return: "Xamsa - Alisher Navoiy"
        """
        return f"{self.title} - {self.author}"
    
    def get_availability_status(self):
        """
        Mavjudlik holati
        """
        if self.copies_available > 0:
            return f"Mavjud ({self.copies_available} nusxa)"
        return "Mavjud emas"
```

### 2.2 Migratsiya

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## 🔧 3. SERIALIZERS YARATISH

### 3.1 Basic Author Serializer

`books/serializers.py`:

```python
from rest_framework import serializers
from .models import Author, Book

class AuthorSerializer(serializers.ModelSerializer):
    """
    Author modelini JSON ga aylantirish
    """
    
    # Custom read-only field - faqat response da ko'rinadi
    # SerializerMethodField - custom metod orqali qiymat olish
    full_name = serializers.SerializerMethodField()
    age = serializers.SerializerMethodField()
    books_count = serializers.SerializerMethodField()
    
    class Meta:
        model = Author
        
        # Qaysi fieldlar JSON da ko'rinadi
        fields = [
            'id',
            'first_name',
            'last_name',
            'full_name',      # Custom field
            'birth_year',
            'age',            # Custom field
            'biography',
            'country',
            'books_count',    # Custom field
            'created_at',
        ]
        
        # Faqat o'qish uchun (create/update da ishlatilmaydi)
        read_only_fields = ['id', 'created_at']
        
        # Qo'shimcha sozlamalar (extra_kwargs)
        # Bu yerda har bir fieldga maxsus sozlamalar beriladi
        extra_kwargs = {
            'first_name': {
                'required': True,  # Majburiy
                'error_messages': {
                    'required': 'Ism kiritilishi shart!',
                    'blank': 'Ism bo\'sh bo\'lmasligi kerak!'
                }
            },
            'last_name': {
                'required': True,
                'error_messages': {
                    'required': 'Familiya kiritilishi shart!',
                }
            },
            'biography': {
                'required': False,  # Ixtiyoriy
                'allow_blank': True,  # Bo'sh bo'lishi mumkin
            }
        }
    
    # Custom field metodlari
    # Bu metodlar SerializerMethodField uchun ishlatiladi
    # Nomi: get_<field_name>
    
    def get_full_name(self, obj):
        """
        To'liq ismni qaytarish
        obj - Author obyekti
        """
        return obj.get_full_name()
    
    def get_age(self, obj):
        """
        Yoshni qaytarish
        """
        return obj.get_age()
    
    def get_books_count(self, obj):
        """
        Muallif yozgan kitoblar soni
        obj.books - related_name orqali barcha kitoblar
        """
        return obj.books.count()
    
    # VALIDATION metodlari
    
    def validate_first_name(self, value):
        """
        Ismni tekshirish
        value - foydalanuvchi kiritgan ism
        """
        # Bo'sh joylarni olib tashlash
        value = value.strip()
        
        # Kamida 2 ta belgi
        if len(value) < 2:
            raise serializers.ValidationError(
                "Ism kamida 2 ta belgidan iborat bo'lishi kerak!"
            )
        
        # Faqat harflar va bo'sh joy
        if not all(char.isalpha() or char.isspace() for char in value):
            raise serializers.ValidationError(
                "Ismda faqat harflar bo'lishi mumkin!"
            )
        
        # Birinchi harfni katta qilish
        value = value.title()
        
        return value
    
    def validate_last_name(self, value):
        """
        Familiyani tekshirish
        """
        value = value.strip()
        
        if len(value) < 2:
            raise serializers.ValidationError(
                "Familiya kamida 2 ta belgidan iborat bo'lishi kerak!"
            )
        
        value = value.title()
        return value
    
    def validate_birth_year(self, value):
        """
        Tug'ilgan yilni tekshirish
        """
        if value:  # Agar berilgan bo'lsa (ixtiyoriy field)
            from datetime import datetime
            current_year = datetime.now().year
            
            # Kelajakda bo'lmasligi kerak
            if value > current_year:
                raise serializers.ValidationError(
                    "Tug'ilgan yil kelajakda bo'lishi mumkin emas!"
                )
            
            # Juda eski bo'lmasligi kerak
            if value < 1800:
                raise serializers.ValidationError(
                    "Tug'ilgan yil 1800 dan kichik bo'lmasligi kerak!"
                )
        
        return value
    
    def validate(self, data):
        """
        Umum validation - barcha fieldlar birgalikda tekshiriladi
        data - dict ko'rinishida barcha ma'lumotlar
        """
        # Ism va familiya bir xil bo'lmasligi kerak
        if 'first_name' in data and 'last_name' in data:
            if data['first_name'].lower() == data['last_name'].lower():
                raise serializers.ValidationError({
                    'last_name': "Ism va familiya bir xil bo'lishi mumkin emas!"
                })
        
        return data
```

### 3.2 Book Serializer (Nested)

```python
class BookSerializer(serializers.ModelSerializer):
    """
    Book modelini JSON ga aylantirish
    """
    
    # Nested serializer - Author ma'lumotlarini ham ko'rsatish
    # read_only=True - faqat GET da ko'rinadi, POST/PUT da yo'q
    author_detail = AuthorSerializer(source='author', read_only=True)
    
    # write_only field - faqat POST/PUT da ishlatiladi, response da ko'rinmaydi
    # author_id ni yuborish orqali muallif tanlanadi
    author_id = serializers.IntegerField(write_only=True)
    
    # Custom fields
    availability_status = serializers.SerializerMethodField()
    genre_display = serializers.CharField(source='get_genre_display', read_only=True)
    
    class Meta:
        model = Book
        fields = [
            'id',
            'title',
            'author_id',        # write_only - faqat POST/PUT da
            'author_detail',    # read_only - faqat GET da
            'isbn',
            'published_year',
            'pages',
            'genre',
            'genre_display',    # Janr nomi (Badiiy adabiyot)
            'description',
            'copies_available',
            'is_available',
            'availability_status',  # Custom field
            'created_at',
            'updated_at',
        ]
        
        read_only_fields = ['id', 'created_at', 'updated_at']
        
        extra_kwargs = {
            'title': {
                'required': True,
                'error_messages': {
                    'required': 'Kitob sarlavhasi kiritilishi shart!',
                    'blank': 'Sarlavha bo\'sh bo\'lmasligi kerak!'
                }
            },
            'isbn': {
                'required': True,
                'error_messages': {
                    'unique': 'Bu ISBN kod allaqachon mavjud!',
                }
            },
            'published_year': {
                'required': True,
            }
        }
    
    def get_availability_status(self, obj):
        """
        Mavjudlik holati
        """
        return obj.get_availability_status()
    
    # VALIDATION
    
    def validate_title(self, value):
        """
        Sarlavhani tekshirish
        """
        value = value.strip()
        
        if len(value) < 3:
            raise serializers.ValidationError(
                "Kitob sarlavhasi kamida 3 ta belgidan iborat bo'lishi kerak!"
            )
        
        # Birinchi harfni katta qilish
        value = value.title()
        
        return value
    
    def validate_isbn(self, value):
        """
        ISBN ni tekshirish
        ISBN - 10 yoki 13 raqamli bo'lishi kerak
        """
        # Bo'sh joylar va chiziqlarni olib tashlash
        value = value.replace('-', '').replace(' ', '').strip()
        
        # Faqat raqamlardan iborat bo'lishi kerak
        if not value.isdigit():
            raise serializers.ValidationError(
                "ISBN faqat raqamlardan iborat bo'lishi kerak!"
            )
        
        # Uzunligi 10 yoki 13 bo'lishi kerak
        if len(value) not in [10, 13]:
            raise serializers.ValidationError(
                "ISBN 10 yoki 13 raqamli bo'lishi kerak!"
            )
        
        return value
    
    def validate_published_year(self, value):
        """
        Chop etilgan yilni tekshirish
        """
        from datetime import datetime
        current_year = datetime.now().year
        
        if value > current_year:
            raise serializers.ValidationError(
                "Chop etilgan yil kelajakda bo'lishi mumkin emas!"
            )
        
        if value < 1000:
            raise serializers.ValidationError(
                "Chop etilgan yil 1000 dan kichik bo'lmasligi kerak!"
            )
        
        return value
    
    def validate_pages(self, value):
        """
        Sahifalar sonini tekshirish
        """
        if value and value < 1:
            raise serializers.ValidationError(
                "Sahifalar soni 1 dan kam bo'lmasligi kerak!"
            )
        return value
    
    def validate_copies_available(self, value):
        """
        Nusxalar sonini tekshirish
        """
        if value < 0:
            raise serializers.ValidationError(
                "Nusxalar soni manfiy bo'lishi mumkin emas!"
            )
        return value
    
    def validate_author_id(self, value):
        """
        Muallif ID sini tekshirish
        """
        # Author database da borligini tekshirish
        try:
            Author.objects.get(id=value)
        except Author.DoesNotExist:
            raise serializers.ValidationError(
                f"ID={value} muallif topilmadi!"
            )
        return value
    
    def validate(self, data):
        """
        Umum validation
        """
        # Agar nusxalar soni 0 bo'lsa, is_available False bo'lishi kerak
        if 'copies_available' in data:
            if data['copies_available'] == 0:
                data['is_available'] = False
            else:
                data['is_available'] = True
        
        # Kitob chop etilgan yil muallifning tug'ilgan yilidan keyin bo'lishi kerak
        if 'author_id' in data and 'published_year' in data:
            author = Author.objects.get(id=data['author_id'])
            if author.birth_year and data['published_year'] < author.birth_year:
                raise serializers.ValidationError({
                    'published_year': "Kitob muallif tug'ilganidan oldin chop etilgan bo'lishi mumkin emas!"
                })
        
        return data
    
    def create(self, validated_data):
        """
        Yangi kitob yaratish
        validated_data - tekshirilgan ma'lumotlar
        """
        # author_id ni author obyektga aylantirish
        author_id = validated_data.pop('author_id')
        author = Author.objects.get(id=author_id)
        
        # Kitob yaratish
        book = Book.objects.create(author=author, **validated_data)
        return book
    
    def update(self, instance, validated_data):
        """
        Kitobni yangilash
        instance - yangilanayotgan kitob
        validated_data - yangi ma'lumotlar
        """
        # Agar author_id berilgan bo'lsa
        if 'author_id' in validated_data:
            author_id = validated_data.pop('author_id')
            instance.author = Author.objects.get(id=author_id)
        
        # Boshqa fieldlarni yangilash
        for attr, value in validated_data.items():
            setattr(instance, attr, value)
        
        instance.save()
        return instance
```

### 3.3 List Serializers (Qisqaroq)

```python
class AuthorListSerializer(serializers.ModelSerializer):
    """
    Ro'yxat uchun qisqa serializer - faqat asosiy ma'lumotlar
    """
    full_name = serializers.SerializerMethodField()
    books_count = serializers.SerializerMethodField()
    
    class Meta:
        model = Author
        fields = ['id', 'full_name', 'country', 'books_count']
    
    def get_full_name(self, obj):
        return obj.get_full_name()
    
    def get_books_count(self, obj):
        return obj.books.count()


class BookListSerializer(serializers.ModelSerializer):
    """
    Kitoblar ro'yxati uchun qisqa serializer
    """
    author_name = serializers.CharField(source='author.get_full_name', read_only=True)
    genre_display = serializers.CharField(source='get_genre_display', read_only=True)
    
    class Meta:
        model = Book
        fields = [
            'id',
            'title',
            'author_name',
            'genre_display',
            'published_year',
            'is_available',
        ]
```

---

## 🎨 4. VIEWS YARATISH

`books/views.py`:

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from .models import Author, Book
from .serializers import (
    AuthorSerializer,
    AuthorListSerializer,
    BookSerializer,
    BookListSerializer
)

# ============ AUTHOR VIEWS ============

@api_view(['GET', 'POST'])
def author_list(request):
    """
    GET: Barcha mualliflar
    POST: Yangi muallif yaratish
    """
    if request.method == 'GET':
        authors = Author.objects.all()
        # List uchun qisqa serializer
        serializer = AuthorListSerializer(authors, many=True)
        return Response(serializer.data)
    
    elif request.method == 'POST':
        # Create uchun to'liq serializer
        serializer = AuthorSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


@api_view(['GET', 'PUT', 'PATCH', 'DELETE'])
def author_detail(request, pk):
    """
    GET: Bitta muallifni ko'rish
    PUT/PATCH: Yangilash
    DELETE: O'chirish
    """
    try:
        author = Author.objects.get(pk=pk)
    except Author.DoesNotExist:
        return Response(
            {'error': f'ID={pk} muallif topilmadi!'},
            status=status.HTTP_404_NOT_FOUND
        )
    
    if request.method == 'GET':
        serializer = AuthorSerializer(author)
        return Response(serializer.data)
    
    elif request.method in ['PUT', 'PATCH']:
        partial = request.method == 'PATCH'
        serializer = AuthorSerializer(author, data=request.data, partial=partial)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    elif request.method == 'DELETE':
        author.delete()
        return Response(
            {'message': 'Muallif o\'chirildi!'},
            status=status.HTTP_204_NO_CONTENT
        )


# ============ BOOK VIEWS ============

@api_view(['GET', 'POST'])
def book_list(request):
    """
    GET: Barcha kitoblar
    POST: Yangi kitob yaratish
    """
    if request.method == 'GET':
        books = Book.objects.all()
        serializer = BookListSerializer(books, many=True)
        return Response(serializer.data)
    
    elif request.method == 'POST':
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


@api_view(['GET', 'PUT', 'PATCH', 'DELETE'])
def book_detail(request, pk):
    """
    GET: Bitta kitobni ko'rish
    PUT/PATCH: Yangilash
    DELETE: O'chirish
    """
    try:
        book = Book.objects.get(pk=pk)
    except Book.DoesNotExist:
        return Response(
            {'error': f'ID={pk} kitob topilmadi!'},
            status=status.HTTP_404_NOT_FOUND
        )
    
    if request.method == 'GET':
        serializer = BookSerializer(book)
        return Response(serializer.data)
    
    elif request.method in ['PUT', 'PATCH']:
        partial = request.method == 'PATCH'
        serializer = BookSerializer(book, data=request.data, partial=partial)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    elif request.method == 'DELETE':
        book.delete()
        return Response(
            {'message': 'Kitob o\'chirildi!'},
            status=status.HTTP_204_NO_CONTENT
        )


# ============ CUSTOM ENDPOINTS ============

@api_view(['GET'])
def author_books(request, pk):
    """
    Muallif yozgan barcha kitoblar
    URL: /api/authors/<id>/books/
    """
    try:
        author = Author.objects.get(pk=pk)
    except Author.DoesNotExist:
        return Response(
            {'error': 'Muallif topilmadi!'},
            status=status.HTTP_404_NOT_FOUND
        )
    
    # related_name orqali barcha kitoblar
    books = author.books.all()
    serializer = BookListSerializer(books, many=True)
    
    return Response({
        'author': author.get_full_name(),
        'books_count': books.count(),
        'books': serializer.data
    })


@api_view(['GET'])
def available_books(request):
    """
    Faqat mavjud kitoblar
    URL: /api/books/available/
    """
    books = Book.objects.filter(is_available=True, copies_available__gt=0)
    serializer = BookListSerializer(books, many=True)
    
    return Response({
        'count': books.count(),
        'books': serializer.data
    })
```

---

## 🌐 5. URL ROUTING

`books/urls.py`:

```python
from django.urls import path
from . import views

app_name = 'books'

urlpatterns = [
    # Authors
    path('authors/', views.author_list, name='author-list'),
    path('authors/<int:pk>/', views.author_detail, name='author-detail'),
    path('authors/<int:pk>/books/', views.author_books, name='author-books'),
    
    # Books
    path('books/', views.book_list, name='book-list'),
    path('books/<int:pk>/', views.book_detail, name='book-detail'),
    path('books/available/', views.available_books, name='available-books'),
]
```

`library_project/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('books.urls')),
]
```

---

## 🛡️ 6. ADMIN PANEL

`books/admin.py`:

```python
from django.contrib import admin
from .models import Author, Book

@admin.register(Author)
class AuthorAdmin(admin.ModelAdmin):
    list_display = ['id', 'first_name', 'last_name', 'country', 'birth_year']
    list_filter = ['country', 'birth_year']
    search_fields = ['first_name', 'last_name']
    readonly_fields = ['created_at']


@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    list_display = [
        'id',
        'title',
        'author',
        'isbn',
        'published_year',
        'genre',
        'is_available',
        'copies_available'
    ]
    list_filter = ['genre', 'is_available', 'published_year']
    search_fields = ['title', 'isbn', 'author__first_name', 'author__last_name']
    list_editable = ['is_available', 'copies_available']
    readonly_fields = ['created_at', 'updated_at']
    
    fieldsets = (
        ('Asosiy Ma\'lumotlar', {
            'fields': ('title', 'author', 'isbn')
        }),
        ('Nashr Ma\'lumotlari', {
            'fields': ('published_year', 'pages', 'genre', 'description')
        }),
        ('Mavjudlik', {
            'fields': ('copies_available', 'is_available')
        }),
        ('Vaqt', {
            'fields': ('created_at', 'updated_at'),
            'classes': ('collapse',)
        }),
    )
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

### 7.2 Test Qilish - cURL

**Muallif yaratish:**

```bash
curl -X POST http://127.0.0.1:8000/api/authors/ \
  -H "Content-Type: application/json" \
  -d '{
    "first_name": "Alisher",
    "last_name": "Navoiy",
    "birth_year": 1441,
    "biography": "O'\''zbek mumtoz adabiyotining buyuk namoyandasi",
    "country": "O'\''zbekiston"
  }'
```

**Kitob yaratish:**

```bash
curl -X POST http://127.0.0.1:8000/api/books/ \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Xamsa",
    "author_id": 1,
    "isbn": "9785990001234",
    "published_year": 1485,
    "pages": 500,
    "genre": "poetry",
    "description": "Besh dostondan iborat asarlar to'\''plami",
    "copies_available": 5
  }'
```

**Barcha kitoblar:**

```bash
curl http://127.0.0.1:8000/api/books/
```

**Muallif kitoblari:**

```bash
curl http://127.0.0.1:8000/api/authors/1/books/
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: SerializerMethodField Qo'shish (Oson)

**Maqsad:** BookSerializer ga yangi custom field qo'shish

**Topshiriq:**
1. `years_since_published` field qo'shing (nechi yil oldin chop etilgan)
2. `is_old_book` field qo'shing (50 yildan eski kitoblar uchun True)
3. API testdan o'tkazing

**Yechim ko'rsatma:**
```python
years_since_published = serializers.SerializerMethodField()

def get_years_since_published(self, obj):
    from datetime import datetime
    current_year = datetime.now().year
    return current_year - obj.published_year
```

---

### 📝 Topshiriq 2: Extra Validation (O'rta)

**Maqsad:** Qo'shimcha validation qoidalari qo'shish

**Topshiriq:**
1. Author uchun: Ism va familiya faqat harflardan iborat bo'lishi
2. Book uchun: Agar sahifalar soni berilgan bo'lsa, 10 dan kam bo'lmasligi
3. Book uchun: Chop etilgan yil muallif vafot etgan yildan katta bo'lmasligi (optional)

**Tekshirish:**
- Noto'g'ri ma'lumot yuborib, xato qaytishini ko'ring
- To'g'ri ma'lumot yuborib, muvaffaqiyatli yaratishni tekshiring

---

### 📝 Topshiriq 3: Publisher Model Qo'shish (Qiyin)

**Maqsad:** Yangi model qo'shib, nested serializer yaratish

**Topshiriq:**
1. `Publisher` modelini yarating (name, country, founded_year)
2. `Book` modeliga `publisher` ForeignKey qo'shing
3. `PublisherSerializer` yarating
4. `BookSerializer` da publisher ma'lumotlarini nested ko'rsating
5. CRUD operatsiyalarini amalga oshiring

**Qo'shimcha:**
- Publisher yozgan barcha kitoblar endpoint: `/api/publishers/<id>/books/`
- Eng ko'p kitob chop etgan nashriyot: `/api/publishers/top/`

---

### 📝 Topshiriq 4: Qidiruv va Filtrlash (O'rta-Qiyin)

**Maqsad:** Qidiruv funksiyasini qo'shish

**Topshiriq:**
1. Kitoblarni sarlavha bo'yicha qidirish: `/api/books/search/?q=xamsa`
2. Kitoblarni janr bo'yicha filtrlash: `/api/books/?genre=poetry`
3. Kitoblarni muallif bo'yicha filtrlash: `/api/books/?author=1`
4. Bir nechta filtrni birga ishlatish

**Maslahat:**
```python
@api_view(['GET'])
def search_books(request):
    query = request.GET.get('q', '')
    books = Book.objects.filter(title__icontains=query)
    # ...
```

---

## 🔗 KEYINGI DARSLAR

✅ **Dars 04 tugadi! ModelSerializer ning barcha imkoniyatlarini o'rgandingiz!**

**Keyingi darsda:**
- Generic Views
- Mixins
- ViewSets

---

## 📚 TEZKOR XULOSA

### ModelSerializer Xususiyatlari

| Xususiyat | Vazifasi | Misol |
|-----------|----------|-------|
| `fields` | Qaysi fieldlar JSON da | `['id', 'title']` |
| `exclude` | Qaysi fieldlar chiqarib tashlanadi | `['password']` |
| `read_only_fields` | Faqat o'qish | `['id', 'created_at']` |
| `extra_kwargs` | Field sozlamalari | `{'title': {'required': True}}` |
| `SerializerMethodField` | Custom field | `get_full_name()` |

### Validation Darajalari

1. **Field-level:** `validate_<field_name>(self, value)`
2. **Object-level:** `validate(self, data)`
3. **Model-level:** Model da `clean()` metodi

### Nested Serializers

```python
# Read-only
author_detail = AuthorSerializer(source='author', read_only=True)

# Write-only
author_id = serializers.IntegerField(write_only=True)
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**
