# 🧩 5-DARS: GENERIC API VIEWS BILAN ISHLASH

## 🎯 Dars Maqsadi

Bu darsda Generic API Views yordamida kod yozishni qisqartirish va professional API yaratishni o'rganasiz. **Blog Platform** loyihasini yaratish orqali barcha generic view'larni amaliyotda ko'rasiz.

**Dars oxirida siz:**
- ✅ Generic API Views nima ekanligini bilasiz
- ✅ Function-based views vs Generic views farqini tushunasiz
- ✅ Barcha generic view turlarini bilasiz (List, Create, Retrieve, Update, Delete)
- ✅ Mixins bilan ishlashni o'rganasiz
- ✅ Custom queryset va permission qo'shishni bilasiz
- ✅ Kod yozishni 10 barobar qisqartirishni o'rganasiz

---

## 💡 Nima Uchun Blog Platform?

**Blog tizimi** - Generic Views uchun eng yaxshi misol:
- 📝 Post yaratish, o'qish, yangilash, o'chirish
- 💬 Comments (izohlar) - nested relationship
- 🏷️ Categories - filtrlash
- 📊 Real-world use case

---

## 📚 Generic Views Nima?

### Oddiy Kod vs Generic Views

**Function-based view (30 qator):**
```python
@api_view(['GET', 'POST'])
def post_list(request):
    if request.method == 'GET':
        posts = Post.objects.all()
        serializer = PostSerializer(posts, many=True)
        return Response(serializer.data)
    
    elif request.method == 'POST':
        serializer = PostSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=201)
        return Response(serializer.errors, status=400)
```

**Generic view (3 qator!):**
```python
class PostListCreateView(generics.ListCreateAPIView):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```

**10 marta kamroq kod! 🚀**

---

## 🚀 1. YANGI LOYIHA YARATISH

### 1.1 Loyiha Sozlash

```bash
# Yangi papka
mkdir blog_api
cd blog_api

# Virtual environment
python -m venv env
env\Scripts\activate  # Windows

# O'rnatish
pip install django djangorestframework

# Loyiha
django-admin startproject blog_project .

# Ilova
python manage.py startapp blog
```

### 1.2 Settings

`blog_project/settings.py`:

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
    
    # Local
    'blog',
]

# DRF Settings
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
}
```

---

## 💾 2. MODELS YARATISH

`blog/models.py`:

```python
from django.db import models
from django.contrib.auth.models import User

class Category(models.Model):
    """
    Blog kategoriyalari
    Masalan: Technology, Lifestyle, Education
    """
    name = models.CharField(
        max_length=100,
        unique=True,
        verbose_name="Kategoriya nomi"
    )
    
    slug = models.SlugField(
        max_length=100,
        unique=True,
        verbose_name="URL uchun nom",
        help_text="URL da ko'rinadi: /category/technology/"
    )
    
    description = models.TextField(
        blank=True,
        null=True,
        verbose_name="Tavsif"
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        verbose_name = "Kategoriya"
        verbose_name_plural = "Kategoriyalar"
        ordering = ['name']
    
    def __str__(self):
        return self.name


class Post(models.Model):
    """
    Blog postlari (maqolalar)
    """
    
    # Status tanlovlari
    STATUS_CHOICES = [
        ('draft', 'Qoralama'),      # Hali nashr qilinmagan
        ('published', 'Nashr qilingan'),  # Ko'rinadi
        ('archived', 'Arxivlangan'),  # Eski postlar
    ]
    
    # Post sarlavhasi (majburiy)
    title = models.CharField(
        max_length=300,
        verbose_name="Sarlavha"
    )
    
    # URL uchun slug
    # SlugField - URL da ishlatish uchun: "my-first-post"
    slug = models.SlugField(
        max_length=300,
        unique=True,
        verbose_name="URL slug"
    )
    
    # Muallif (ForeignKey - User modeli)
    # User - Django'ning built-in model
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='posts',
        verbose_name="Muallif"
    )
    
    # Kategoriya (ixtiyoriy)
    category = models.ForeignKey(
        Category,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='posts',
        verbose_name="Kategoriya"
    )
    
    # Qisqacha mazmun (excerpt)
    excerpt = models.TextField(
        max_length=500,
        blank=True,
        null=True,
        verbose_name="Qisqacha",
        help_text="Post haqida qisqacha (list da ko'rinadi)"
    )
    
    # To'liq matn
    content = models.TextField(
        verbose_name="Kontent",
        help_text="Post ning to'liq matni"
    )
    
    # Rasm URL (ixtiyoriy)
    # URLField - rasm URL manzili
    featured_image = models.URLField(
        blank=True,
        null=True,
        verbose_name="Asosiy rasm",
        help_text="https://example.com/image.jpg"
    )
    
    # Status
    status = models.CharField(
        max_length=10,
        choices=STATUS_CHOICES,
        default='draft',
        verbose_name="Holat"
    )
    
    # Ko'rishlar soni
    views_count = models.IntegerField(
        default=0,
        verbose_name="Ko'rishlar soni"
    )
    
    # Vaqt maydonlari
    published_at = models.DateTimeField(
        blank=True,
        null=True,
        verbose_name="Nashr qilingan vaqt"
    )
    
    created_at = models.DateTimeField(
        auto_now_add=True,
        verbose_name="Yaratilgan"
    )
    
    updated_at = models.DateTimeField(
        auto_now=True,
        verbose_name="Yangilangan"
    )
    
    class Meta:
        verbose_name = "Post"
        verbose_name_plural = "Postlar"
        ordering = ['-created_at']  # Eng yangilari birinchi
        indexes = [
            models.Index(fields=['slug']),
            models.Index(fields=['status']),
        ]
    
    def __str__(self):
        return self.title
    
    def increment_views(self):
        """
        Ko'rishlar sonini 1 ga oshirish
        """
        self.views_count += 1
        self.save(update_fields=['views_count'])


class Comment(models.Model):
    """
    Postlarga izohlar (comments)
    """
    
    # Qaysi postga izoh
    post = models.ForeignKey(
        Post,
        on_delete=models.CASCADE,
        related_name='comments',
        verbose_name="Post"
    )
    
    # Kim yozgan (ixtiyoriy - anonim bo'lishi mumkin)
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='comments',
        verbose_name="Muallif"
    )
    
    # Anonim ism (agar User bo'lmasa)
    author_name = models.CharField(
        max_length=100,
        blank=True,
        null=True,
        verbose_name="Ism (agar ro'yxatdan o'tmagan bo'lsa)"
    )
    
    # Izoh matni
    content = models.TextField(
        verbose_name="Izoh"
    )
    
    # Tasdiqlangan yoki yo'q (moderation)
    is_approved = models.BooleanField(
        default=False,
        verbose_name="Tasdiqlangan",
        help_text="Admin tomonidan tasdiqlangan"
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        verbose_name = "Izoh"
        verbose_name_plural = "Izohlar"
        ordering = ['-created_at']
    
    def __str__(self):
        return f"{self.author or self.author_name}: {self.content[:50]}"
```

### 2.2 Migratsiya

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## 🔧 3. SERIALIZERS

`blog/serializers.py`:

```python
from rest_framework import serializers
from django.contrib.auth.models import User
from .models import Category, Post, Comment

# ============ CATEGORY SERIALIZER ============

class CategorySerializer(serializers.ModelSerializer):
    """
    Kategoriya serializer
    """
    posts_count = serializers.SerializerMethodField()
    
    class Meta:
        model = Category
        fields = ['id', 'name', 'slug', 'description', 'posts_count', 'created_at']
        read_only_fields = ['id', 'created_at']
    
    def get_posts_count(self, obj):
        """Kategoryadagi postlar soni"""
        return obj.posts.filter(status='published').count()


# ============ USER SERIALIZER ============

class UserSerializer(serializers.ModelSerializer):
    """
    User serializer (muallif ma'lumotlari)
    """
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'first_name', 'last_name']
        read_only_fields = ['id']


# ============ COMMENT SERIALIZERS ============

class CommentSerializer(serializers.ModelSerializer):
    """
    Izoh serializer
    """
    author_username = serializers.CharField(
        source='author.username',
        read_only=True
    )
    
    class Meta:
        model = Comment
        fields = [
            'id',
            'post',
            'author',
            'author_username',
            'author_name',
            'content',
            'is_approved',
            'created_at'
        ]
        read_only_fields = ['id', 'created_at', 'is_approved']
    
    def validate(self, data):
        """
        Agar author bo'lmasa, author_name majburiy
        """
        if not data.get('author') and not data.get('author_name'):
            raise serializers.ValidationError({
                'author_name': 'Agar ro\'yxatdan o\'tmagan bo\'lsangiz, ism kiriting!'
            })
        return data


# ============ POST SERIALIZERS ============

class PostListSerializer(serializers.ModelSerializer):
    """
    Post ro'yxati uchun - qisqa ma'lumot
    """
    author_name = serializers.CharField(
        source='author.username',
        read_only=True
    )
    
    category_name = serializers.CharField(
        source='category.name',
        read_only=True
    )
    
    comments_count = serializers.SerializerMethodField()
    
    class Meta:
        model = Post
        fields = [
            'id',
            'title',
            'slug',
            'author_name',
            'category_name',
            'excerpt',
            'featured_image',
            'status',
            'views_count',
            'comments_count',
            'published_at',
            'created_at'
        ]
    
    def get_comments_count(self, obj):
        """Tasdiqlangan izohlar soni"""
        return obj.comments.filter(is_approved=True).count()


class PostDetailSerializer(serializers.ModelSerializer):
    """
    Post batafsil - to'liq ma'lumot
    """
    # Nested serializers
    author = UserSerializer(read_only=True)
    category = CategorySerializer(read_only=True)
    comments = CommentSerializer(many=True, read_only=True)
    
    # Write-only fields (create/update uchun)
    author_id = serializers.IntegerField(write_only=True, required=False)
    category_id = serializers.IntegerField(write_only=True, required=False)
    
    # Custom fields
    reading_time = serializers.SerializerMethodField()
    
    class Meta:
        model = Post
        fields = [
            'id',
            'title',
            'slug',
            'author',
            'author_id',
            'category',
            'category_id',
            'excerpt',
            'content',
            'featured_image',
            'status',
            'views_count',
            'reading_time',
            'comments',
            'published_at',
            'created_at',
            'updated_at'
        ]
        read_only_fields = ['id', 'views_count', 'created_at', 'updated_at']
    
    def get_reading_time(self, obj):
        """
        O'qish vaqti (daqiqalarda)
        O'rtacha 200 so'z/daqiqa
        """
        words = len(obj.content.split())
        minutes = words // 200
        return max(1, minutes)  # Kamida 1 daqiqa
    
    def validate_slug(self, value):
        """Slug unikal bo'lishi kerak"""
        # Update paytida o'zini tekshirmaslik
        instance = getattr(self, 'instance', None)
        if instance:
            if Post.objects.exclude(pk=instance.pk).filter(slug=value).exists():
                raise serializers.ValidationError("Bu slug allaqachon ishlatilgan!")
        else:
            if Post.objects.filter(slug=value).exists():
                raise serializers.ValidationError("Bu slug allaqachon ishlatilgan!")
        return value
    
    def create(self, validated_data):
        """Yangi post yaratish"""
        # author_id ni author obyektga aylantirish
        author_id = validated_data.pop('author_id', None)
        category_id = validated_data.pop('category_id', None)
        
        if author_id:
            validated_data['author_id'] = author_id
        if category_id:
            validated_data['category_id'] = category_id
        
        # Agar status='published' bo'lsa, published_at ni set qilish
        if validated_data.get('status') == 'published' and not validated_data.get('published_at'):
            from django.utils import timezone
            validated_data['published_at'] = timezone.now()
        
        return Post.objects.create(**validated_data)
    
    def update(self, instance, validated_data):
        """Post yangilash"""
        # author_id va category_id ni olish
        author_id = validated_data.pop('author_id', None)
        category_id = validated_data.pop('category_id', None)
        
        if author_id:
            instance.author_id = author_id
        if category_id:
            instance.category_id = category_id
        
        # Agar status draft -> published o'zgarsa, published_at ni set qilish
        if instance.status != 'published' and validated_data.get('status') == 'published':
            from django.utils import timezone
            instance.published_at = timezone.now()
        
        # Boshqa fieldlar
        for attr, value in validated_data.items():
            setattr(instance, attr, value)
        
        instance.save()
        return instance
```

---

## 🎨 4. GENERIC VIEWS

### 4.1 Barcha Generic View Turlari

`blog/views.py`:

```python
from rest_framework import generics, status
from rest_framework.response import Response
from rest_framework.decorators import api_view
from .models import Category, Post, Comment
from .serializers import (
    CategorySerializer,
    PostListSerializer,
    PostDetailSerializer,
    CommentSerializer
)

# ============================================
# GENERIC VIEWS TURLARI
# ============================================

# 1. ListAPIView - Faqat ro'yxatni ko'rish (GET)
# 2. CreateAPIView - Faqat yaratish (POST)
# 3. RetrieveAPIView - Faqat bitta elementni ko'rish (GET)
# 4. UpdateAPIView - Faqat yangilash (PUT/PATCH)
# 5. DestroyAPIView - Faqat o'chirish (DELETE)

# KOMBINATSIYALAR:
# 6. ListCreateAPIView - Ro'yxat va yaratish (GET, POST)
# 7. RetrieveUpdateAPIView - Ko'rish va yangilash (GET, PUT, PATCH)
# 8. RetrieveDestroyAPIView - Ko'rish va o'chirish (GET, DELETE)
# 9. RetrieveUpdateDestroyAPIView - Ko'rish, yangilash va o'chirish (GET, PUT, PATCH, DELETE)


# ============ CATEGORY VIEWS ============

class CategoryListCreateView(generics.ListCreateAPIView):
    """
    GET:  Barcha kategoriyalar
    POST: Yangi kategoriya yaratish
    
    Generic View: ListCreateAPIView
    - GET va POST metodlarini avtomatik boshqaradi
    - queryset va serializer_class ni belgilashimiz kifoya
    """
    queryset = Category.objects.all()
    serializer_class = CategorySerializer


class CategoryDetailView(generics.RetrieveUpdateDestroyAPIView):
    """
    GET:    Bitta kategoriyani ko'rish
    PUT:    To'liq yangilash
    PATCH:  Qisman yangilash
    DELETE: O'chirish
    
    Generic View: RetrieveUpdateDestroyAPIView
    - Barcha CRUD operatsiyalari (Read, Update, Delete)
    - 3 qator kod bilan to'liq CRUD!
    """
    queryset = Category.objects.all()
    serializer_class = CategorySerializer


# ============ POST VIEWS ============

class PostListView(generics.ListAPIView):
    """
    Faqat ro'yxatni ko'rish (GET)
    
    Generic View: ListAPIView
    - Faqat GET metodi
    - Yaratish (POST) yo'q
    """
    serializer_class = PostListSerializer
    
    def get_queryset(self):
        """
        Custom queryset - faqat nashr qilingan postlar
        
        get_queryset() - querysetni dinamik filtrlash uchun
        Bu yerda faqat published postlarni qaytaramiz
        """
        return Post.objects.filter(status='published').select_related(
            'author', 'category'
        )


class PostCreateView(generics.CreateAPIView):
    """
    Faqat yaratish (POST)
    
    Generic View: CreateAPIView
    - Faqat POST metodi
    - Ro'yxatni ko'rish (GET) yo'q
    """
    queryset = Post.objects.all()
    serializer_class = PostDetailSerializer


class PostDetailView(generics.RetrieveUpdateDestroyAPIView):
    """
    Bitta postni ko'rish, yangilash, o'chirish
    
    Generic View: RetrieveUpdateDestroyAPIView
    """
    queryset = Post.objects.all()
    serializer_class = PostDetailSerializer
    lookup_field = 'slug'  # ID o'rniga slug ishlatamiz
    
    def retrieve(self, request, *args, **kwargs):
        """
        GET metodi - ko'rishlar sonini oshirish
        
        retrieve() - GET request handle qiluvchi metod
        Biz uni override qilib, ko'rishlar sonini oshiramiz
        """
        instance = self.get_object()
        instance.increment_views()  # Ko'rishlar soni +1
        serializer = self.get_serializer(instance)
        return Response(serializer.data)


# ============ COMMENT VIEWS ============

class CommentListCreateView(generics.ListCreateAPIView):
    """
    Barcha izohlar va yangi izoh yaratish
    """
    queryset = Comment.objects.filter(is_approved=True)  # Faqat tasdiqlangan
    serializer_class = CommentSerializer


class CommentDetailView(generics.RetrieveUpdateDestroyAPIView):
    """
    Bitta izohni boshqarish
    """
    queryset = Comment.objects.all()
    serializer_class = CommentSerializer


# ============ CUSTOM ENDPOINTS ============

class PostsByCategoryView(generics.ListAPIView):
    """
    Kategoriya bo'yicha postlar
    URL: /api/categories/<slug>/posts/
    """
    serializer_class = PostListSerializer
    
    def get_queryset(self):
        """
        URL dan category slug ni olish
        """
        category_slug = self.kwargs['slug']
        return Post.objects.filter(
            category__slug=category_slug,
            status='published'
        ).select_related('author', 'category')


class PostCommentListView(generics.ListAPIView):
    """
    Bitta postning barcha izohlari
    URL: /api/posts/<slug>/comments/
    """
    serializer_class = CommentSerializer
    
    def get_queryset(self):
        """
        URL dan post slug ni olish
        """
        post_slug = self.kwargs['slug']
        return Comment.objects.filter(
            post__slug=post_slug,
            is_approved=True
        )


class DraftPostsView(generics.ListAPIView):
    """
    Qoralama postlar (draft)
    URL: /api/posts/drafts/
    """
    queryset = Post.objects.filter(status='draft')
    serializer_class = PostListSerializer


class PopularPostsView(generics.ListAPIView):
    """
    Eng ko'p ko'rilgan postlar
    URL: /api/posts/popular/
    """
    serializer_class = PostListSerializer
    
    def get_queryset(self):
        """
        Ko'rishlar soni bo'yicha tartiblash
        """
        return Post.objects.filter(
            status='published'
        ).order_by('-views_count')[:10]  # Top 10
```

---

## 🌐 5. URL ROUTING

`blog/urls.py`:

```python
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    # ============ CATEGORIES ============
    # ListCreateAPIView - GET va POST
    path(
        'categories/',
        views.CategoryListCreateView.as_view(),
        name='category-list-create'
    ),
    
    # RetrieveUpdateDestroyAPIView - GET, PUT, PATCH, DELETE
    path(
        'categories/<int:pk>/',
        views.CategoryDetailView.as_view(),
        name='category-detail'
    ),
    
    # Kategoriya bo'yicha postlar
    path(
        'categories/<slug:slug>/posts/',
        views.PostsByCategoryView.as_view(),
        name='category-posts'
    ),
    
    # ============ POSTS ============
    # ListAPIView - faqat GET
    path(
        'posts/',
        views.PostListView.as_view(),
        name='post-list'
    ),
    
    # CreateAPIView - faqat POST
    path(
        'posts/create/',
        views.PostCreateView.as_view(),
        name='post-create'
    ),
    
    # Draft postlar
    path(
        'posts/drafts/',
        views.DraftPostsView.as_view(),
        name='post-drafts'
    ),
    
    # Popular postlar
    path(
        'posts/popular/',
        views.PopularPostsView.as_view(),
        name='post-popular'
    ),
    
    # RetrieveUpdateDestroyAPIView - GET, PUT, PATCH, DELETE
    # slug bilan ishlash
    path(
        'posts/<slug:slug>/',
        views.PostDetailView.as_view(),
        name='post-detail'
    ),
    
    # Post izohlari
    path(
        'posts/<slug:slug>/comments/',
        views.PostCommentListView.as_view(),
        name='post-comments'
    ),
    
    # ============ COMMENTS ============
    path(
        'comments/',
        views.CommentListCreateView.as_view(),
        name='comment-list-create'
    ),
    
    path(
        'comments/<int:pk>/',
        views.CommentDetailView.as_view(),
        name='comment-detail'
    ),
]
```

`blog_project/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('blog.urls')),
]
```

---

## 🛡️ 6. ADMIN PANEL

`blog/admin.py`:

```python
from django.contrib import admin
from .models import Category, Post, Comment

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['id', 'name', 'slug']
    prepopulated_fields = {'slug': ('name',)}


@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = [
        'id',
        'title',
        'author',
        'category',
        'status',
        'views_count',
        'published_at'
    ]
    list_filter = ['status', 'category', 'created_at']
    search_fields = ['title', 'content']
    prepopulated_fields = {'slug': ('title',)}
    list_editable = ['status']
    date_hierarchy = 'created_at'


@admin.register(Comment)
class CommentAdmin(admin.ModelAdmin):
    list_display = ['id', 'post', 'author', 'is_approved', 'created_at']
    list_filter = ['is_approved', 'created_at']
    list_editable = ['is_approved']
```

**Superuser:**

```bash
python manage.py createsuperuser
```

---

## ✅ 7. TESTING

### 7.1 Server

```bash
python manage.py runserver
```

### 7.2 Test - cURL

**Kategoriya yaratish:**

```bash
curl -X POST http://127.0.0.1:8000/api/categories/ \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Technology",
    "slug": "technology",
    "description": "Tech news and tutorials"
  }'
```

**Post yaratish:**

```bash
curl -X POST http://127.0.0.1:8000/api/posts/create/ \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Django REST Framework Tutorial",
    "slug": "drf-tutorial",
    "author_id": 1,
    "category_id": 1,
    "excerpt": "Learn DRF step by step",
    "content": "This is a comprehensive guide...",
    "status": "published"
  }'
```

---

## 📚 8. GENERIC VIEWS JADVALI

| Generic View | Metodlar | Vazifasi |
|-------------|----------|----------|
| `ListAPIView` | GET | Faqat ro'yxat |
| `CreateAPIView` | POST | Faqat yaratish |
| `RetrieveAPIView` | GET | Faqat bitta element |
| `UpdateAPIView` | PUT, PATCH | Faqat yangilash |
| `DestroyAPIView` | DELETE | Faqat o'chirish |
| `ListCreateAPIView` | GET, POST | Ro'yxat + yaratish |
| `RetrieveUpdateAPIView` | GET, PUT, PATCH | Ko'rish + yangilash |
| `RetrieveDestroyAPIView` | GET, DELETE | Ko'rish + o'chirish |
| `RetrieveUpdateDestroyAPIView` | GET, PUT, PATCH, DELETE | Barcha CRUD (Read, Update, Delete) |

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Search Endpoint (Oson) ⭐

**Maqsad:** Postlarni qidirish

Create: `SearchPostsView` - postlarni title bo'yicha qidirish

**URL:** `/api/posts/search/?q=django`

**Ko'rsatma:**
```python
class SearchPostsView(generics.ListAPIView):
    serializer_class = PostListSerializer
    
    def get_queryset(self):
        query = self.request.GET.get('q', '')
        return Post.objects.filter(title__icontains=query)
```

---

### 📝 Topshiriq 2: Pagination Qo'shish (Oson) ⭐

**Maqsad:** Ro'yxatlarga pagination

**Topshiriq:**
1. Custom pagination class yarating (5 ta post per page)
2. `PostListView` ga qo'shing
3. Test qiling

**Ko'rsatma:**
```python
from rest_framework.pagination import PageNumberPagination

class PostPagination(PageNumberPagination):
    page_size = 5
```

---

### 📝 Topshiriq 3: Filtering by Author (O'rta) ⭐⭐

**Maqsad:** Muallif bo'yicha filtrlash

**Topshiriq:**
1. `PostsByAuthorView` yarating
2. URL: `/api/authors/<username>/posts/`
3. Faqat o'sha muallifning postlari

---

### 📝 Topshiriq 4: Like System (Qiyin) ⭐⭐⭐

**Maqsad:** Like tizimini qo'shish

**Topshiriq:**
1. `Like` modelini yarating (User, Post, created_at)
2. `LikeCreateView` - POST da like qo'shish
3. `PostDetailSerializer` da `likes_count` qo'shing
4. Bir user bitta postga faqat 1 marta like qo'yishi mumkin

---

### 📝 Topshiriq 5: Tags System (Qiyin) ⭐⭐⭐

**Maqsad:** Tag tizimi (ManyToMany)

**Topshiriq:**
1. `Tag` modelini yarating
2. `Post` modeliga `tags` ManyToManyField qo'shing
3. Serializer da tags ko'rsating
4. `/api/tags/<slug>/posts/` - tag bo'yicha postlar

---

## 🔗 KEYINGI DARSLAR

✅ **Dars 05 tugadi! Generic Views bilan professional API yaratishni o'rgandingiz!**

**Keyingi darsda:**
- ViewSets
- Routers
- Actions

---

## 📚 TEZKOR XULOSA

### Generic Views vs Function-Based Views

| Xususiyat | FBV | Generic Views |
|-----------|-----|---------------|
| Kod uzunligi | 30-50 qator | 3-5 qator |
| O'rganish qiyinligi | Oson | O'rtacha |
| Customization | ✅✅✅ | ✅✅ |
| **Tavsiya** | Oddiy API | Standard CRUD |

### Override Qilinadigan Metodlar

```python
get_queryset()       # Custom queryset
get_serializer_class()  # Dinamik serializer
get_object()         # Obyektni olish
perform_create()     # Create jarayonini customize
perform_update()   # Update jarayonini customize
perform_destroy()    # Delete jarayonini customize
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**
