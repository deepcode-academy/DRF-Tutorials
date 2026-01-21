# 🧩 6-DARS: VIEWSETS VA ROUTERS BILAN ISHLASH

## 🎯 Dars Maqsadi

Bu darsda ViewSets va Routers yordamida API yaratishni avtomatlashtirishni o'rganasiz. **E-commerce Store (Online Do'kon)** loyihasini yaratish orqali ViewSets'ning barcha imkoniyatlarini va custom actions yaratishni amaliyotda ko'rasiz.

**Dars oxirida siz:**
- ✅ ViewSets nima va qanday ishlashini bilasiz
- ✅ Router bilan URL'larni avtomatik yaratishni o'rganasiz
- ✅ Custom actions (@action decorator) yaratishni bilasiz
- ✅ ModelViewSet, ReadOnlyModelViewSet farqini tushunasiz
- ✅ ViewSet vs Generic Views qachon qaysi birini ishlatishni bilasiz
- ✅ Kod yozishni 20 barobar qisqartirishni o'rganasiz

---

## 💡 Nima Uchun E-commerce Store?

**Online do'kon** - ViewSets uchun eng yaxshi misol:
- 🛍️ Products - mahsulotlar (CRUD + custom actions)
- 🏷️ Categories - kategoriyalar
- 🛒 Cart - savatcha (custom actions kerak!)
- ⭐ Reviews - sharh va reyting
- 📊 Real-world use case

---

## 📚 ViewSets Nima?

### Kod Taqqoslash

**Generic Views (10+ qator):**
```python
class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer

class ProductDetailView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
```

**ViewSet (3 qator!):**
```python
class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
```

**Bir ViewSet = Barcha CRUD operatsiyalari! 🚀**

---

## 🚀 1. YANGI LOYIHA YARATISH

### 1.1 Loyiha Sozlash

```bash
# Yangi papka
mkdir ecommerce_api
cd ecommerce_api

# Virtual environment
python -m venv env
env\Scripts\activate  # Windows

# O'rnatish
pip install django djangorestframework

# Loyiha
django-admin startproject ecommerce_project .

# Ilova
python manage.py startapp store
```

### 1.2 Settings

`ecommerce_project/settings.py`:

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
    'store',
]

REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
}
```

---

## 💾 2. MODELS YARATISH

`store/models.py`:

```python
from django.db import models
from django.core.validators import MinValueValidator, MaxValueValidator
from django.contrib.auth.models import User

class Category(models.Model):
    """
    Mahsulot kategoriyalari
    Masalan: Electronics, Clothing, Books
    """
    name = models.CharField(
        max_length=100,
        unique=True,
        verbose_name="Kategoriya nomi"
    )
    
    slug = models.SlugField(
        max_length=100,
        unique=True,
        verbose_name="URL slug"
    )
    
    description = models.TextField(
        blank=True,
        verbose_name="Tavsif"
    )
    
    image = models.URLField(
        blank=True,
        null=True,
        verbose_name="Kategoriya rasmi"
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        verbose_name = "Kategoriya"
        verbose_name_plural = "Kategoriyalar"
        ordering = ['name']
    
    def __str__(self):
        return self.name


class Product(models.Model):
    """
    Mahsulotlar modeli
    """
    # Mahsulot nomi
    name = models.CharField(
        max_length=300,
        verbose_name="Mahsulot nomi"
    )
    
    # URL uchun slug
    slug = models.SlugField(
        max_length=300,
        unique=True,
        verbose_name="URL slug"
    )
    
    # Kategoriya
    category = models.ForeignKey(
        Category,
        on_delete=models.CASCADE,
        related_name='products',
        verbose_name="Kategoriya"
    )
    
    # Tavsif
    description = models.TextField(
        verbose_name="Tavsif"
    )
    
    # Narx (DecimalField - pullar uchun)
    # max_digits=10 - jami 10 ta raqam
    # decimal_places=2 - verguldan keyin 2 ta raqam (99.99)
    price = models.DecimalField(
        max_digits=10,
        decimal_places=2,
        verbose_name="Narx",
        validators=[MinValueValidator(0)]
    )
    
    # Chegirma (0-100%)
    discount_percentage = models.IntegerField(
        default=0,
        validators=[
            MinValueValidator(0),
            MaxValueValidator(100)
        ],
        verbose_name="Chegirma %"
    )
    
    # Omborda mavjud miqdor
    stock = models.IntegerField(
        default=0,
        validators=[MinValueValidator(0)],
        verbose_name="Omborda",
        help_text="Nechta dona omborda bor"
    )
    
    # Mahsulot rasmi
    image = models.URLField(
        blank=True,
        null=True,
        verbose_name="Mahsulot rasmi"
    )
    
    # Faol (sotuvda) yoki yo'q
    is_active = models.BooleanField(
        default=True,
        verbose_name="Faol",
        help_text="Sotuvda bormi?"
    )
    
    # Featured (ommabop mahsulot)
    is_featured = models.BooleanField(
        default=False,
        verbose_name="Ommabop",
        help_text="Asosiy sahifada ko'rsatilsinmi?"
    )
    
    # Ko'rishlar soni
    views_count = models.IntegerField(
        default=0,
        verbose_name="Ko'rishlar"
    )
    
    # Sotilgan miqdor
    sold_count = models.IntegerField(
        default=0,
        verbose_name="Sotilgan"
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name = "Mahsulot"
        verbose_name_plural = "Mahsulotlar"
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['slug']),
            models.Index(fields=['category']),
            models.Index(fields=['is_active']),
        ]
    
    def __str__(self):
        return self.name
    
    def get_discounted_price(self):
        """
        Chegirmali narx
        Masalan: 100$ - 20% = 80$
        """
        if self.discount_percentage > 0:
            discount = (self.price * self.discount_percentage) / 100
            return self.price - discount
        return self.price
    
    def is_in_stock(self):
        """
        Omborda bormi?
        """
        return self.stock > 0
    
    def increment_views(self):
        """
        Ko'rishlar sonini +1 qilish
        """
        self.views_count += 1
        self.save(update_fields=['views_count'])


class Review(models.Model):
    """
    Mahsulot sharhlari va reytinglari
    """
    # Qaysi mahsulot
    product = models.ForeignKey(
        Product,
        on_delete=models.CASCADE,
        related_name='reviews',
        verbose_name="Mahsulot"
    )
    
    # Kim yozgan
    user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='reviews',
        verbose_name="Foydalanuvchi"
    )
    
    # Reyting (1-5 yulduz)
    rating = models.IntegerField(
        validators=[
            MinValueValidator(1),
            MaxValueValidator(5)
        ],
        verbose_name="Reyting",
        help_text="1 dan 5 gacha"
    )
    
    # Sharh matni (ixtiyoriy)
    comment = models.TextField(
        blank=True,
        verbose_name="Sharh"
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        verbose_name = "Sharh"
        verbose_name_plural = "Sharhlar"
        ordering = ['-created_at']
        # Bir user bitta mahsulotga faqat 1 marta sharh yozishi mumkin
        unique_together = ['product', 'user']
    
    def __str__(self):
        return f"{self.user.username} - {self.product.name} ({self.rating}⭐)"
```

### 2.2 Migratsiya

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## 🔧 3. SERIALIZERS

`store/serializers.py`:

```python
from rest_framework import serializers
from django.contrib.auth.models import User
from .models import Category, Product, Review

# ============ CATEGORY SERIALIZER ============

class CategorySerializer(serializers.ModelSerializer):
    """
    Kategoriya serializer
    """
    products_count = serializers.SerializerMethodField()
    
    class Meta:
        model = Category
        fields = ['id', 'name', 'slug', 'description', 'image', 'products_count', 'created_at']
        read_only_fields = ['id', 'created_at']
    
    def get_products_count(self, obj):
        """Kategoryadagi mahsulotlar soni"""
        return obj.products.filter(is_active=True).count()


# ============ REVIEW SERIALIZER ============

class ReviewSerializer(serializers.ModelSerializer):
    """
    Sharh serializer
    """
    user_name = serializers.CharField(source='user.username', read_only=True)
    
    class Meta:
        model = Review
        fields = ['id', 'product', 'user', 'user_name', 'rating', 'comment', 'created_at']
        read_only_fields = ['id', 'user', 'created_at']
    
    def validate_rating(self, value):
        """Reyting 1-5 oralig'ida bo'lishi kerak"""
        if value < 1 or value > 5:
            raise serializers.ValidationError("Reyting 1 dan 5 gacha bo'lishi kerak!")
        return value


# ============ PRODUCT SERIALIZERS ============

class ProductListSerializer(serializers.ModelSerializer):
    """
    Mahsulotlar ro'yxati uchun - qisqa ma'lumot
    """
    category_name = serializers.CharField(source='category.name', read_only=True)
    discounted_price = serializers.SerializerMethodField()
    in_stock = serializers.SerializerMethodField()
    average_rating = serializers.SerializerMethodField()
    
    class Meta:
        model = Product
        fields = [
            'id',
            'name',
            'slug',
            'category_name',
            'price',
            'discount_percentage',
            'discounted_price',
            'stock',
            'in_stock',
            'image',
            'is_featured',
            'average_rating',
            'views_count',
            'sold_count',
        ]
    
    def get_discounted_price(self, obj):
        """Chegirmali narx"""
        return float(obj.get_discounted_price())
    
    def get_in_stock(self, obj):
        """Omborda bormi?"""
        return obj.is_in_stock()
    
    def get_average_rating(self, obj):
        """O'rtacha reyting"""
        reviews = obj.reviews.all()
        if reviews.exists():
            total = sum(review.rating for review in reviews)
            return round(total / reviews.count(), 1)
        return 0


class ProductDetailSerializer(serializers.ModelSerializer):
    """
    Mahsulot batafsil - to'liq ma'lumot
    """
    category = CategorySerializer(read_only=True)
    category_id = serializers.IntegerField(write_only=True)
    
    reviews = ReviewSerializer(many=True, read_only=True)
    reviews_count = serializers.SerializerMethodField()
    average_rating = serializers.SerializerMethodField()
    
    discounted_price = serializers.SerializerMethodField()
    savings = serializers.SerializerMethodField()
    
    class Meta:
        model = Product
        fields = [
            'id',
            'name',
            'slug',
            'category',
            'category_id',
            'description',
            'price',
            'discount_percentage',
            'discounted_price',
            'savings',
            'stock',
            'image',
            'is_active',
            'is_featured',
            'views_count',
            'sold_count',
            'reviews',
            'reviews_count',
            'average_rating',
            'created_at',
            'updated_at',
        ]
        read_only_fields = ['id', 'views_count', 'sold_count', 'created_at', 'updated_at']
    
    def get_discounted_price(self, obj):
        """Chegirmali narx"""
        return float(obj.get_discounted_price())
    
    def get_savings(self, obj):
        """Qancha tejaydi"""
        if obj.discount_percentage > 0:
            return float(obj.price - obj.get_discounted_price())
        return 0
    
    def get_reviews_count(self, obj):
        """Sharhlar soni"""
        return obj.reviews.count()
    
    def get_average_rating(self, obj):
        """O'rtacha reyting"""
        reviews = obj.reviews.all()
        if reviews.exists():
            total = sum(review.rating for review in reviews)
            return round(total / reviews.count(), 1)
        return 0
```

---

## 🎨 4. VIEWSETS YARATISH

`store/views.py`:

```python
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from django.db.models import Q
from .models import Category, Product, Review
from .serializers import (
    CategorySerializer,
    ProductListSerializer,
    ProductDetailSerializer,
    ReviewSerializer
)

# ============================================
# VIEWSET TURLARI
# ============================================

# 1. ModelViewSet - To'liq CRUD (list, retrieve, create, update, delete)
# 2. ReadOnlyModelViewSet - Faqat o'qish (list, retrieve)
# 3. ViewSet - Base class, metodlarni o'zingiz yozasiz


# ============ CATEGORY VIEWSET ============

class CategoryViewSet(viewsets.ModelViewSet):
    """
    Category CRUD API
    
    ModelViewSet - avtomatik barcha CRUD operatsiyalari:
    - list (GET /api/categories/)
    - retrieve (GET /api/categories/{id}/)
    - create (POST /api/categories/)
    - update (PUT /api/categories/{id}/)
    - partial_update (PATCH /api/categories/{id}/)
    - destroy (DELETE /api/categories/{id}/)
    """
    queryset = Category.objects.all()
    serializer_class = CategorySerializer
    lookup_field = 'slug'  # ID o'rniga slug ishlatish
    
    # Custom action - kategoriya mahsulotlari
    @action(detail=True, methods=['get'])
    def products(self, request, slug=None):
        """
        Kategoriya mahsulotlari
        
        URL: /api/categories/{slug}/products/
        
        @action parametrlari:
        - detail=True: Bitta obyekt uchun (/categories/{slug}/products/)
        - detail=False: Umum (/categories/all-products/)
        - methods=['get']: Qaysi HTTP metodlar
        """
        category = self.get_object()
        products = category.products.filter(is_active=True)
        serializer = ProductListSerializer(products, many=True)
        
        return Response({
            'category': category.name,
            'count': products.count(),
            'products': serializer.data
        })


# ============ PRODUCT VIEWSET ============

class ProductViewSet(viewsets.ModelViewSet):
    """
    Product CRUD API + custom actions
    """
    queryset = Product.objects.filter(is_active=True).select_related('category')
    
    def get_serializer_class(self):
        """
        Dinamik serializer tanlash
        
        - list action uchun: ProductListSerializer (qisqa)
        - retrieve action uchun: ProductDetailSerializer (to'liq)
        - boshqalar uchun: ProductDetailSerializer
        """
        if self.action == 'list':
            return ProductListSerializer
        return ProductDetailSerializer
    
    def get_queryset(self):
        """
        Custom queryset - filtrlash
        """
        queryset = Product.objects.filter(is_active=True).select_related('category')
        
        # URL parametrlarni olish
        # Masalan: /api/products/?category=electronics&min_price=100
        category = self.request.query_params.get('category', None)
        min_price = self.request.query_params.get('min_price', None)
        max_price = self.request.query_params.get('max_price', None)
        
        # Filtrlash
        if category:
            queryset = queryset.filter(category__slug=category)
        
        if min_price:
            queryset = queryset.filter(price__gte=min_price)
        
        if max_price:
            queryset = queryset.filter(price__lte=max_price)
        
        return queryset
    
    def retrieve(self, request, *args, **kwargs):
        """
        Bitta mahsulotni olish
        Ko'rishlar sonini +1 qilish
        """
        instance = self.get_object()
        instance.increment_views()
        serializer = self.get_serializer(instance)
        return Response(serializer.data)
    
    # ============ CUSTOM ACTIONS ============
    
    @action(detail=False, methods=['get'])
    def featured(self, request):
        """
        Ommabop mahsulotlar
        
        URL: /api/products/featured/
        
        detail=False - umum action (barcha mahsulotlar uchun)
        """
        featured_products = Product.objects.filter(
            is_active=True,
            is_featured=True
        ).select_related('category')[:10]
        
        serializer = ProductListSerializer(featured_products, many=True)
        return Response(serializer.data)
    
    @action(detail=False, methods=['get'])
    def bestsellers(self, request):
        """
        Eng ko'p sotilgan mahsulotlar
        
        URL: /api/products/bestsellers/
        """
        bestsellers = Product.objects.filter(
            is_active=True
        ).order_by('-sold_count')[:20]
        
        serializer = ProductListSerializer(bestsellers, many=True)
        return Response(serializer.data)
    
    @action(detail=False, methods=['get'])
    def on_sale(self, request):
        """
        Chegirmadagi mahsulotlar
        
        URL: /api/products/on_sale/
        """
        on_sale = Product.objects.filter(
            is_active=True,
            discount_percentage__gt=0
        ).select_related('category')
        
        serializer = ProductListSerializer(on_sale, many=True)
        return Response(serializer.data)
    
    @action(detail=False, methods=['get'])
    def search(self, request):
        """
        Qidiruv
        
        URL: /api/products/search/?q=laptop
        """
        query = request.query_params.get('q', '')
        
        if not query:
            return Response(
                {'error': 'Qidiruv so\'zi kiritilmagan!'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        # Nom va tavsifda qidiruv
        products = Product.objects.filter(
            Q(name__icontains=query) | Q(description__icontains=query),
            is_active=True
        ).select_related('category')
        
        serializer = ProductListSerializer(products, many=True)
        
        return Response({
            'query': query,
            'count': products.count(),
            'results': serializer.data
        })
    
    @action(detail=True, methods=['post'])
    def add_review(self, request, pk=None):
        """
        Mahsulotga sharh qo'shish
        
        URL: /api/products/{id}/add_review/
        POST: {"rating": 5, "comment": "Juda yaxshi!"}
        
        detail=True - bitta mahsulot uchun action
        methods=['post'] - faqat POST
        """
        product = self.get_object()
        
        # Serializer yaratish
        serializer = ReviewSerializer(data=request.data)
        
        if serializer.is_valid():
            # User va productni set qilish
            serializer.save(
                user=request.user,
                product=product
            )
            return Response(
                serializer.data,
                status=status.HTTP_201_CREATED
            )
        
        return Response(
            serializer.errors,
            status=status.HTTP_400_BAD_REQUEST
        )
    
    @action(detail=True, methods=['post'])
    def purchase(self, request, pk=None):
        """
        Mahsulot sotib olish (soddalashtirilgan)
        
        URL: /api/products/{id}/purchase/
        POST: {"quantity": 2}
        """
        product = self.get_object()
        quantity = request.data.get('quantity', 1)
        
        # Omborda borligini tekshirish
        if product.stock < quantity:
            return Response(
                {'error': f'Omborda faqat {product.stock} dona bor!'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        # Stockni kamaytirish
        product.stock -= quantity
        product.sold_count += quantity
        product.save()
        
        total_price = float(product.get_discounted_price()) * quantity
        
        return Response({
            'message': 'Xarid muvaffaqiyatli!',
            'product': product.name,
            'quantity': quantity,
            'unit_price': float(product.get_discounted_price()),
            'total_price': total_price,
            'remaining_stock': product.stock
        })


# ============ REVIEW VIEWSET ============

class ReviewViewSet(viewsets.ModelViewSet):
    """
    Review CRUD API
    """
    queryset = Review.objects.all().select_related('product', 'user')
    serializer_class = ReviewSerializer
    
    def perform_create(self, serializer):
        """
        Create vaqtida user'ni avtomatik set qilish
        
        perform_create() - create operatsiyasi vaqtida chaqiriladi
        """
        serializer.save(user=self.request.user)
    
    @action(detail=False, methods=['get'])
    def my_reviews(self, request):
        """
        Foydalanuvchining barcha sharhlari
        
        URL: /api/reviews/my_reviews/
        """
        my_reviews = Review.objects.filter(user=request.user)
        serializer = self.get_serializer(my_reviews, many=True)
        return Response(serializer.data)
```

---

## 🌐 5. ROUTERS VA URL

`store/urls.py`:

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from . import views

# ============ ROUTER YARATISH ============

# DefaultRouter - standart URL pattern yaratadi
router = DefaultRouter()

# ViewSetlarni register qilish
# register(r'prefix', ViewSetClass, basename='url-name')

router.register(r'categories', views.CategoryViewSet, basename='category')
router.register(r'products', views.ProductViewSet, basename='product')
router.register(r'reviews', views.ReviewViewSet, basename='review')

# Router avtomatik quyidagi URL'larni yaratadi:
#
# CATEGORIES:
# - GET    /api/categories/                    - list
# - POST   /api/categories/                    - create
# - GET    /api/categories/{slug}/             - retrieve
# - PUT    /api/categories/{slug}/             - update
# - PATCH  /api/categories/{slug}/             - partial_update
# - DELETE /api/categories/{slug}/             - destroy
# - GET    /api/categories/{slug}/products/    - custom action
#
# PRODUCTS:
# - GET    /api/products/                      - list
# - POST   /api/products/                      - create
# - GET    /api/products/{id}/                 - retrieve
# - PUT    /api/products/{id}/                 - update
# - PATCH  /api/products/{id}/                 - partial_update
# - DELETE /api/products/{id}/                 - destroy
# - GET    /api/products/featured/             - custom action
# - GET    /api/products/bestsellers/          - custom action
# - GET    /api/products/on_sale/              - custom action
# - GET    /api/products/search/               - custom action
# - POST   /api/products/{id}/add_review/      - custom action
# - POST   /api/products/{id}/purchase/        - custom action
#
# REVIEWS:
# - GET    /api/reviews/                       - list
# - POST   /api/reviews/                       - create
# - GET    /api/reviews/{id}/                  - retrieve
# - PUT    /api/reviews/{id}/                  - update
# - PATCH  /api/reviews/{id}/                  - partial_update
# - DELETE /api/reviews/{id}/                  - destroy
# - GET    /api/reviews/my_reviews/            - custom action

app_name = 'store'

urlpatterns = [
    # Router URL'larini qo'shish
    path('', include(router.urls)),
]
```

`ecommerce_project/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('store.urls')),
]
```

---

## 🛡️ 6. ADMIN PANEL

`store/admin.py`:

```python
from django.contrib import admin
from .models import Category, Product, Review

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['id', 'name', 'slug']
    prepopulated_fields = {'slug': ('name',)}


@admin.register(Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = [
        'id',
        'name',
        'category',
        'price',
        'discount_percentage',
        'stock',
        'is_active',
        'is_featured',
        'sold_count',
        'views_count'
    ]
    list_filter = ['category', 'is_active', 'is_featured']
    search_fields = ['name', 'description']
    prepopulated_fields = {'slug': ('name',)}
    list_editable = ['is_active', 'is_featured', 'stock']


@admin.register(Review)
class ReviewAdmin(admin.ModelAdmin):
    list_display = ['id', 'product', 'user', 'rating', 'created_at']
    list_filter = ['rating', 'created_at']
```

---

## ✅ 7. TESTING

```bash
python manage.py runserver
```

### Barcha URL'lar:

```
http://127.0.0.1:8000/api/
http://127.0.0.1:8000/api/categories/
http://127.0.0.1:8000/api/products/
http://127.0.0.1:8000/api/products/featured/
http://127.0.0.1:8000/api/products/bestsellers/
http://127.0.0.1:8000/api/products/on_sale/
http://127.0.0.1:8000/api/products/search/?q=laptop
http://127.0.0.1:8000/api/reviews/
```

---

## 📚 8. VIEWSET TURLARI

| ViewSet | Metodlar | Qachon ishlatish? |
|---------|----------|-------------------|
| `ModelViewSet` | list, retrieve, create, update, delete | To'liq CRUD kerak bo'lsa |
| `ReadOnlyModelViewSet` | list, retrieve | Faqat o'qish (create/update yo'q) |
| `ViewSet` | Custom | O'zingiz metodlar yozasiz |

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Wishlist Action (Oson) ⭐

**Maqsad:** Sevimlilar ro'yxati

**Topshiriq:**
1. `@action` yarating: `add_to_wishlist`
2. POST /api/products/{id}/add_to_wishlist/
3. Responseda "Sevimliga qo'shildi" message

---

### 📝 Topshiriq 2: Price Range Filter (O'rta) ⭐⭐

**Maqsad:** Narx oralig'i bo'yicha filtrlash

**Topshiriq:**
1. `get_queryset()` da narx oralig'ini qo'shing
2. `/api/products/?min_price=100&max_price=500`

---

### 📝 Topshiriq 3: Stats Action (O'rta) ⭐⭐

**Maqsad:** Statistika ko'rsatish

**Topshiriq:**
1.`@action` yarating: `stats` (detail=False)
2. Jami mahsulotlar, o'rtacha narx, eng qimmat/arzon
3. `/api/products/stats/`

---

### 📝 Topshiriq 4: Related Products (Qiyin) ⭐⭐⭐

**Maqsad:** O'xshash mahsulotlar

**Topshiriq:**
1. `@action`: `related` (detail=True)
2.O'sha kategoriyadan boshqa mahsulotlar
3. `/api/products/{id}/related/`

---

### 📝 Topshiriq 5: Bulk Delete (Qiyin) ⭐⭐⭐

**Maqsad:** Bir nechta mahsulotni o'chirish

**Topshiriq:**
1. `@action`: `bulk_delete` (detail=False, methods=['post'])
2. POST: `{"ids": [1, 2, 3]}`
3. Barcha mahsulotlarni o'chirish

---

## 🔗 KEYINGI DARS

✅ **Dars 06 tugadi! ViewSets va Routers bilan professional API yaratdingiz!**

**Keyingi darsda:**
- Authentication
- Permissions
- Token-based auth

---

## 📚 XULOSA

### ViewSet vs Generic Views

| Xususiyat | Generic Views | ViewSet |
|-----------|---------------|---------|
| URL yaratish | Manual | Avtomatik (Router) |
| Kod uzunligi | O'rtacha | Qisqa |
| Customization | ✅✅✅ | ✅✅ |
| Custom actions | ❌ | ✅✅✅ (`@action`) |
| **Tavsiya** | Oddiy API | Standard CRUD + actions |

### @action Decorator

```python
@action(detail=True, methods=['post'])
def custom_action(self, request, pk=None):
    # detail=True - /api/products/{pk}/custom_action/
    # detail=False - /api/products/custom_action/
    # methods - HTTP metodlar ro'yxati
    pass
```

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**
