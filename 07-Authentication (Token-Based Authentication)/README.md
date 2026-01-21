# 🧩 7-DARS: TOKEN-BASED AUTHENTICATION

## 🎯 Dars Maqsadi

Bu darsda Token-based Authentication yordamida API ni himoyalashni o'rganasiz. **Social Media API** loyihasini yaratish orqali Register, Login, Logout, User profile va authentication bilan ishlashni amaliyotda ko'rasiz.

**Dars oxirida siz:**
- ✅ Token Authentication nima ekanligini bilasiz
- ✅ User registration (ro'yxatdan o'tish) qilishni o'rganasiz
- ✅ Login/Logout qilishni bilasiz
- ✅ Token bilan API himoyalashni o'rganasiz
- ✅ User profil bilan ishlashni bilasiz
- ✅ Custom permissions yaratishni o'rganasiz

---

## 💡 Nima Uchun Social Media API?

**Ijtimoiy tarmoq** - Authentication uchun eng yaxshi misol:
- 👤 User registration va login
- 📝 Post yaratish (faqat login qilganlar)
- 💬 Comment (faqat o'z postini o'chirish)
- 👥 Follow/Unfollow tizimi
- 📊 Real-world use case

---

## 📚 Token Authentication Nima?

### Session vs Token

**Session-based (Traditional):**
```
User → Login → Server creates session → Session ID in cookie
Problem: Mobile apps uchun mos emas, CORS muammolari
```

**Token-based (Modern API):**
```
User → Login → Server generates token → Token in Authorization header
Afzalligi: Mobile-friendly, Stateless, CORS yo'q
```

---

## 🚀 1. YANGI LOYIHA YARATISH

### 1.1 Loyiha Sozlash

```bash
mkdir social_api
cd social_api
python -m venv env
env\Scripts\activate  # Windows

pip install django djangorestframework

django-admin startproject social_project .
python manage.py startapp users
python manage.py startapp posts
```

### 1.2 Settings

`social_project/settings.py`:

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
    'rest_framework.authtoken',  # Token authentication uchun
    
    # Local apps
    'users',
    'posts',
]

# DRF Settings
REST_FRAMEWORK = {
    # Authentication classes
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
    # Permission classes (global)
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ],
}
```

### 1.3 Migratsiya

```bash
python manage.py migrate
```

---

## 💾 2. MODELS YARATISH

### 2.1 User Profile Model

`users/models.py`:

```python
from django.db import models
from django.contrib.auth.models import User
from django.db.models.signals import post_save
from django.dispatch import receiver

class UserProfile(models.Model):
    """
    User profil modeli
    Django'ning User modeliga qo'shimcha ma'lumotlar
    """
    # OneToOneField - Har bir User uchun bitta Profile
    user = models.OneToOneField(
        User,
        on_delete=models.CASCADE,
        related_name='profile',
        verbose_name="Foydalanuvchi"
    )
    
    # Bio (qisqa tavsif)
    bio = models.TextField(
        max_length=500,
        blank=True,
        verbose_name="Bio",
        help_text="O'zingiz haqingizda qisqacha"
    )
    
    # Avatar (profil rasmi)
    avatar = models.URLField(
        blank=True,
        null=True,
        verbose_name="Avatar"
    )
    
    # Tug'ilgan kun
    birth_date = models.DateField(
        blank=True,
        null=True,
        verbose_name="Tug'ilgan kun"
    )
    
    # Lokatsiya
    location = models.CharField(
        max_length=100,
        blank=True,
        verbose_name="Joylashuv"
    )
    
    # Website
    website = models.URLField(
        blank=True,
        null=True,
        verbose_name="Website"
    )
    
    # Followers (kuzatuvchilar)
    followers = models.ManyToManyField(
        User,
        related_name='following',
        blank=True,
        verbose_name="Kuzatuvchilar"
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name = "Profil"
        verbose_name_plural = "Profillar"
    
    def __str__(self):
        return f"{self.user.username}'s profile"
    
    def followers_count(self):
        """Kuzatuvchilar soni"""
        return self.followers.count()
    
    def following_count(self):
        """Kuzatayotganlar soni"""
        return self.user.following.count()


# Signal - User yaratilganda avtomatik Profile yaratish
@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    """
    User yaratilganda avtomatik UserProfile yaratish
    
    Signal - voqea sodir bo'lganda avtomatik ishlaydigan funksiya
    post_save - obyekt saqlanganidan keyin
    """
    if created:
        UserProfile.objects.create(user=instance)


@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    """User saqlanganida Profile ham saqlash"""
    instance.profile.save()
```

### 2.2 Post Model

`posts/models.py`:

```python
from django.db import models
from django.contrib.auth.models import User

class Post(models.Model):
    """
    Post (ijtimoiy tarmoq posti) modeli
    """
    # Post muallifi
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='posts',
        verbose_name="Muallif"
    )
    
    # Post matni
    content = models.TextField(
        verbose_name="Kontent",
        help_text="Post matni"
    )
    
    # Rasm (ixtiyoriy)
    image = models.URLField(
        blank=True,
        null=True,
        verbose_name="Rasm"
    )
    
    # Likes (yoqtirishlar)
    # ManyToManyField - ko'p User ko'p Post ga like qo'yishi mumkin
    likes = models.ManyToManyField(
        User,
        related_name='liked_posts',
        blank=True,
        verbose_name="Yoqtirishlar"
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name = "Post"
        verbose_name_plural = "Postlar"
        ordering = ['-created_at']
    
    def __str__(self):
        return f"{self.author.username}: {self.content[:50]}"
    
    def likes_count(self):
        """Like'lar soni"""
        return self.likes.count()


class Comment(models.Model):
    """
    Postlarga izohlar
    """
    # Qaysi post
    post = models.ForeignKey(
        Post,
        on_delete=models.CASCADE,
        related_name='comments',
        verbose_name="Post"
    )
    
    # Kim yozgan
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='comments',
        verbose_name="Muallif"
    )
    
    # Izoh matni
    content = models.TextField(
        verbose_name="Izoh"
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        verbose_name = "Izoh"
        verbose_name_plural = "Izohlar"
        ordering = ['-created_at']
    
    def __str__(self):
        return f"{self.author.username} on {self.post.id}"
```

### 2.3 Migratsiya

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## 🔧 3. SERIALIZERS

### 3.1 User Serializers

`users/serializers.py`:

```python
from rest_framework import serializers
from django.contrib.auth.models import User
from django.contrib.auth.password_validation import validate_password
from .models import UserProfile

class UserProfileSerializer(serializers.ModelSerializer):
    """
    User profile serializer
    """
    followers_count = serializers.SerializerMethodField()
    following_count = serializers.SerializerMethodField()
    
    class Meta:
        model = UserProfile
        fields = [
            'bio',
            'avatar',
            'birth_date',
            'location',
            'website',
            'followers_count',
            'following_count',
        ]
    
    def get_followers_count(self, obj):
        return obj.followers_count()
    
    def get_following_count(self, obj):
        return obj.following_count()


class UserSerializer(serializers.ModelSerializer):
    """
    User serializer (public ma'lumotlar)
    """
    profile = UserProfileSerializer(read_only=True)
    posts_count = serializers.SerializerMethodField()
    
    class Meta:
        model = User
        fields = [
            'id',
            'username',
            'email',
            'first_name',
            'last_name',
            'profile',
            'posts_count',
            'date_joined',
        ]
        read_only_fields = ['id', 'date_joined']
    
    def get_posts_count(self, obj):
        return obj.posts.count()


class RegisterSerializer(serializers.ModelSerializer):
    """
    Ro'yxatdan o'tish uchun serializer
    """
    # Password fieldlari (write_only)
    password = serializers.CharField(
        write_only=True,
        required=True,
        validators=[validate_password],
        style={'input_type': 'password'}
    )
    
    password2 = serializers.CharField(
        write_only=True,
        required=True,
        style={'input_type': 'password'},
        label="Parolni tasdiqlang"
    )
    
    class Meta:
        model = User
        fields = [
            'username',
            'email',
            'password',
            'password2',
            'first_name',
            'last_name',
        ]
        extra_kwargs = {
            'first_name': {'required': True},
            'last_name': {'required': True},
            'email': {'required': True},
        }
    
    def validate(self, attrs):
        """
        Parollar bir xilligini tekshirish
        """
        if attrs['password'] != attrs['password2']:
            raise serializers.ValidationError({
                'password': "Parollar bir xil emas!"
            })
        return attrs
    
    def create(self, validated_data):
        """
        Yangi user yaratish
        """
        # password2 ni o'chirish (kerak emas)
        validated_data.pop('password2')
        
        # User yaratish
        user = User.objects.create_user(
            username=validated_data['username'],
            email=validated_data['email'],
            first_name=validated_data['first_name'],
            last_name=validated_data['last_name'],
            password=validated_data['password']
        )
        
        return user


class ChangePasswordSerializer(serializers.Serializer):
    """
    Parolni o'zgartirish
    """
    old_password = serializers.CharField(
        required=True,
        write_only=True,
        style={'input_type': 'password'}
    )
    
    new_password = serializers.CharField(
        required=True,
        write_only=True,
        validators=[validate_password],
        style={'input_type': 'password'}
    )
    
    new_password2 = serializers.CharField(
        required=True,
        write_only=True,
        style={'input_type': 'password'}
    )
    
    def validate(self, attrs):
        """Yangi parollar bir xilligini tekshirish"""
        if attrs['new_password'] != attrs['new_password2']:
            raise serializers.ValidationError({
                'new_password': "Parollar bir xil emas!"
            })
        return attrs
```

### 3.2 Post Serializers

`posts/serializers.py`:

```python
from rest_framework import serializers
from .models import Post, Comment
from users.serializers import UserSerializer

class CommentSerializer(serializers.ModelSerializer):
    """
    Comment serializer
    """
    author = UserSerializer(read_only=True)
    
    class Meta:
        model = Comment
        fields = ['id', 'post', 'author', 'content', 'created_at']
        read_only_fields = ['id', 'author', 'created_at']


class PostSerializer(serializers.ModelSerializer):
    """
    Post serializer
    """
    author = UserSerializer(read_only=True)
    comments = CommentSerializer(many=True, read_only=True)
    likes_count = serializers.SerializerMethodField()
    is_liked = serializers.SerializerMethodField()
    comments_count = serializers.SerializerMethodField()
    
    class Meta:
        model = Post
        fields = [
            'id',
            'author',
            'content',
            'image',
            'likes_count',
            'is_liked',
            'comments',
            'comments_count',
            'created_at',
            'updated_at',
        ]
        read_only_fields = ['id', 'author', 'created_at', 'updated_at']
    
    def get_likes_count(self, obj):
        return obj.likes_count()
    
    def get_is_liked(self, obj):
        """Hozirgi user like qo'yganmi?"""
        request = self.context.get('request')
        if request and request.user.is_authenticated:
            return obj.likes.filter(id=request.user.id).exists()
        return False
    
    def get_comments_count(self, obj):
        return obj.comments.count()
```

---

## 🎨 4. AUTHENTICATION VIEWS

`users/views.py`:

```python
from rest_framework import status, generics, viewsets
from rest_framework.decorators import action, api_view, permission_classes
from rest_framework.response import Response
from rest_framework.authtoken.models import Token
from rest_framework.permissions import IsAuthenticated, AllowAny
from django.contrib.auth.models import User
from django.contrib.auth import authenticate
from .models import UserProfile
from .serializers import (
    RegisterSerializer,
    UserSerializer,
    UserProfileSerializer,
    ChangePasswordSerializer
)

# ============ REGISTRATION ============

class RegisterView(generics.CreateAPIView):
    """
    User ro'yxatdan o'tkazish
    
    POST /api/register/
    {
        "username": "john",
        "email": "john@example.com",
        "password": "securepass123",
        "password2": "securepass123",
        "first_name": "John",
        "last_name": "Doe"
    }
    """
    queryset = User.objects.all()
    serializer_class = RegisterSerializer
    permission_classes = [AllowAny]  # Hamma ro'yxatdan o'tishi mumkin
    
    def create(self, request, *args, **kwargs):
        """
        User yaratish va token qaytarish
        """
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        user = serializer.save()
        
        # Token yaratish
        token, created = Token.objects.get_or_create(user=user)
        
        return Response({
            'user': UserSerializer(user).data,
            'token': token.key,
            'message': 'Ro\'yxatdan o\'tdingiz!'
        }, status=status.HTTP_201_CREATED)


# ============ LOGIN ============

@api_view(['POST'])
@permission_classes([AllowAny])
def login_view(request):
    """
    Login qilish
    
    POST /api/login/
    {
        "username": "john",
        "password": "securepass123"
    }
    """
    username = request.data.get('username')
    password = request.data.get('password')
    
    if not username or not password:
        return Response(
            {'error': 'Username va password kiritilishi shart!'},
            status=status.HTTP_400_BAD_REQUEST
        )
    
    # Authenticate - username va parol to'g'rimi?
    user = authenticate(username=username, password=password)
    
    if user:
        # Token olish yoki yaratish
        token, created = Token.objects.get_or_create(user=user)
        
        return Response({
            'user': UserSerializer(user).data,
            'token': token.key,
            'message': 'Login muvaffaqiyatli!'
        })
    else:
        return Response(
            {'error': 'Username yoki parol noto\'g\'ri!'},
            status=status.HTTP_401_UNAUTHORIZED
        )


# ============ LOGOUT ============

@api_view(['POST'])
@permission_classes([IsAuthenticated])
def logout_view(request):
    """
    Logout qilish (tokenni o'chirish)
    
    POST /api/logout/
    Headers: Authorization: Token <your-token>
    """
    # Tokenni o'chirish
    request.user.auth_token.delete()
    
    return Response({
        'message': 'Logout muvaffaqiyatli!'
    })


# ============ USER VIEWSET ============

class UserViewSet(viewsets.ReadOnlyModelViewSet):
    """
    User CRUD (faqat o'qish)
    
    list: Barcha userlar
    retrieve: Bitta user
    """
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = [IsAuthenticated]
    
    @action(detail=False, methods=['get'])
    def me(self, request):
        """
        Hozirgi user ma'lumotlari
        
        GET /api/users/me/
        """
        serializer = self.get_serializer(request.user)
        return Response(serializer.data)
    
    @action(detail=False, methods=['put', 'patch'])
    def update_profile(self, request):
        """
        Profilni yangilash
        
        PUT/PATCH /api/users/update_profile/
        """
        profile = request.user.profile
        serializer = UserProfileSerializer(
            profile,
            data=request.data,
            partial=True
        )
        
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        
        return Response(
            serializer.errors,
            status=status.HTTP_400_BAD_REQUEST
        )
    
    @action(detail=False, methods=['post'])
    def change_password(self, request):
        """
        Parolni o'zgartirish
        
        POST /api/users/change_password/
        {
            "old_password": "oldpass",
            "new_password": "newpass123",
            "new_password2": "newpass123"
        }
        """
        serializer = ChangePasswordSerializer(data=request.data)
        
        if serializer.is_valid():
            # Eski parolni tekshirish
            if not request.user.check_password(serializer.data['old_password']):
                return Response(
                    {'old_password': 'Eski parol noto\'g\'ri!'},
                    status=status.HTTP_400_BAD_REQUEST
                )
            
            # Yangi parolni o'rnatish
            request.user.set_password(serializer.data['new_password'])
            request.user.save()
            
            return Response({'message': 'Parol o\'zgartirildi!'})
        
        return Response(
            serializer.errors,
            status=status.HTTP_400_BAD_REQUEST
        )
    
    @action(detail=True, methods=['post'])
    def follow(self, request, pk=None):
        """
        Userni kuzatish (follow)
        
        POST /api/users/{id}/follow/
        """
        user_to_follow = self.get_object()
        
        # O'zini kuzata olmasligi
        if user_to_follow == request.user:
            return Response(
                {'error': 'O\'zingizni kuzata olmaysiz!'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        # Follow/Unfollow
        if user_to_follow.profile.followers.filter(id=request.user.id).exists():
            # Unfollow
            user_to_follow.profile.followers.remove(request.user)
            return Response({'message': 'Unfollowed!'})
        else:
            # Follow
            user_to_follow.profile.followers.add(request.user)
            return Response({'message': 'Followed!'})
```

---

## 🌐 5. POST VIEWS VA PERMISSIONS

`posts/views.py`:

```python
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticatedOrReadOnly, IsAuthenticated
from .models import Post, Comment
from .serializers import PostSerializer, CommentSerializer

# Custom Permission
class IsAuthorOrReadOnly(permissions.BasePermission):
    """
    Custom permission - faqat muallif o'zgartirishi/o'chirishi mumkin
    """
    def has_object_permission(self, request, view, obj):
        # GET, HEAD, OPTIONS - hamma ko'rishi mumkin
        if request.method in permissions.SAFE_METHODS:
            return True
        
        # PUT, PATCH, DELETE - faqat muallif
        return obj.author == request.user


class PostViewSet(viewsets.ModelViewSet):
    """
    Post CRUD
    """
    queryset = Post.objects.all().select_related('author')
    serializer_class = PostSerializer
    permission_classes = [IsAuthenticatedOrReadOnly, IsAuthorOrReadOnly]
    
    def perform_create(self, serializer):
        """
        Post yaratishda avtomatik author ni set qilish
        """
        serializer.save(author=self.request.user)
    
    @action(detail=True, methods=['post'])
    def like(self, request, pk=None):
        """
        Postga like/unlike qo'yish
        
        POST /api/posts/{id}/like/
        """
        post = self.get_object()
        
        # Like/Unlike
        if post.likes.filter(id=request.user.id).exists():
            # Unlike
            post.likes.remove(request.user)
            return Response({'message': 'Unliked!'})
        else:
            # Like
            post.likes.add(request.user)
            return Response({'message': 'Liked!'})
    
    @action(detail=False, methods=['get'])
    def my_posts(self, request):
        """
        Mening postlarim
        
        GET /api/posts/my_posts/
        """
        my_posts = Post.objects.filter(author=request.user)
        serializer = self.get_serializer(my_posts, many=True)
        return Response(serializer.data)


class CommentViewSet(viewsets.ModelViewSet):
    """
    Comment CRUD
    """
    queryset = Comment.objects.all()
    serializer_class = CommentSerializer
    permission_classes = [IsAuthenticatedOrReadOnly, IsAuthorOrReadOnly]
    
    def perform_create(self, serializer):
        serializer.save(author=self.request.user)
```

---

## 📡 6. URLS

`users/urls.py`:

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from . import views

router = DefaultRouter()
router.register(r'users', views.UserViewSet, basename='user')

urlpatterns = [
    path('register/', views.RegisterView.as_view(), name='register'),
    path('login/', views.login_view, name='login'),
    path('logout/', views.logout_view, name='logout'),
    path('', include(router.urls)),
]
```

`posts/urls.py`:

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from . import views

router = DefaultRouter()
router.register(r'posts', views.PostViewSet, basename='post')
router.register(r'comments', views.CommentViewSet, basename='comment')

urlpatterns = [
    path('', include(router.urls)),
]
```

`social_project/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('users.urls')),
    path('api/', include('posts.urls')),
]
```

---

## ✅ 7. TESTING

```bash
python manage.py runserver
```

### Register:

```bash
curl -X POST http://127.0.0.1:8000/api/register/ \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john",
    "email": "john@example.com",
    "password": "securepass123",
    "password2": "securepass123",
    "first_name": "John",
    "last_name": "Doe"
  }'
```

### Login:

```bash
curl -X POST http://127.0.0.1:8000/api/login/ \
  -H "Content-Type: application/json" \
  -d '{"username": "john", "password": "securepass123"}'
```

### Post yaratish (token bilan):

```bash
curl -X POST http://127.0.0.1:8000/api/posts/ \
  -H "Authorization: Token YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{"content": "My first post!"}'
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: Email Verification ⭐⭐
Register qilganda email verification

### 📝 Topshiriq 2: Forgot Password ⭐⭐
Parol unutish funksiyasi

### 📝 Topshiriq 3: User Search ⭐
Username bo'yicha qidirish

### 📝 Topshiriq 4: Following Feed ⭐⭐⭐
Faqat kuzatayotgan userlar postlari

### 📝 Topshiriq 5: Block User ⭐⭐⭐
Userni bloklash/unbloklash

---

**© 2024 Deepcode Academy**
