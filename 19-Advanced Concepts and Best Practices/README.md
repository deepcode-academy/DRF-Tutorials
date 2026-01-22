# 🎓 19-DARS: ADVANCED CONCEPTS VA BEST PRACTICES

## 🎯 Dars Maqsadi

Bu darsda Django REST Framework'da **Advanced Concepts** va **Best Practices** - professional API development uchun zarur bo'lgan ilg'or tushunchalar va eng yaxshi amaliyotlarni o'rganasiz.

**Dars oxirida siz:**
- ✅ API Versioning strategies
- ✅ Auto-generated documentation (Swagger/OpenAPI)
- ✅ CORS configuration
- ✅ Custom exception handling
- ✅ API rate limiting strategies
- ✅ Security best practices
- ✅ Code organization patterns
- ✅ Monitoring va logging
- ✅ Production checklist

---

## 📚 Oldingi Darsdan Kerakli Bilimlar

Bu darsni boshlashdan oldin quyidagilar tayyor bo'lishi kerak:

- [x] Barcha oldingi darslar
- [x] Production deployment experience
- [x] REST API principles

> **Eslatma:** Bu dars - kursmizning yakunlovchi qismi!

---

## 🔍 1. API VERSIONING

### 1.1 Nega Versioning Kerak?

```
Without Versioning:
API changes → Breaking changes → Angry users 😡

With Versioning:
API v1 (old clients) → Still works ✅
API v2 (new clients) → New features ✅
```

### 1.2 Versioning Strategies

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| **URL Path** | `/api/v1/tasks/` | Clear, Cacheable | URL clutter |
| **Query Parameter** | `/api/tasks/?version=1` | Flexible | Not standard |
| **Header** | `Accept: application/vnd.api+json; version=1` | Clean URLs | Hidden |
| **Hostname** | `v1.api.example.com` | Isolation | Infrastructure |

### 1.3 Implementation (URL Path)

`myproject/settings.py`:

```python
REST_FRAMEWORK = {
    # ... other settings ...
    
    # Versioning
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.URLPathVersioning',
    'DEFAULT_VERSION': 'v1',
    'ALLOWED_VERSIONS': ['v1', 'v2'],
    'VERSION_PARAM': 'version',
}
```

`myproject/urls.py`:

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from tasks.views import TaskViewSetV1, TaskViewSetV2

# V1 Router
router_v1 = DefaultRouter()
router_v1.register(r'tasks', TaskViewSetV1, basename='task')

# V2 Router
router_v2 = DefaultRouter()
router_v2.register(r'tasks', TaskViewSetV2, basename='task')

urlpatterns = [
    # API v1
    path('api/v1/', include(router_v1.urls)),
    
    # API v2
    path('api/v2/', include(router_v2.urls)),
]
```

`tasks/views.py`:

```python
from rest_framework import viewsets
from .models import Task
from .serializers import TaskSerializerV1, TaskSerializerV2

class TaskViewSetV1(viewsets.ModelViewSet):
    """
    API Version 1 - Deprecated
    """
    queryset = Task.objects.all()
    serializer_class = TaskSerializerV1
    
    def list(self, request, *args, **kwargs):
        # Add deprecation warning
        response = super().list(request, *args, **kwargs)
        response['Warning'] = '299 - "API v1 is deprecated. Please migrate to v2"'
        return response


class TaskViewSetV2(viewsets.ModelViewSet):
    """
    API Version 2 - Current
    
    New features:
    - Additional fields
    - Better performance
    - Improved filtering
    """
    queryset = Task.objects.select_related('owner').prefetch_related('tags')
    serializer_class = TaskSerializerV2
    
    # V2 specific features
    def get_queryset(self):
        queryset = super().get_queryset()
        # V2 specific optimizations
        return queryset
```

---

## 📖 2. AUTO-GENERATED DOCUMENTATION

### 2.1 drf-spectacular Setup

```bash
pip install drf-spectacular
```

`myproject/settings.py`:

```python
INSTALLED_APPS = [
    # ...
    'drf_spectacular',
]

REST_FRAMEWORK = {
    # ...
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}

# Spectacular settings
SPECTACULAR_SETTINGS = {
    'TITLE': 'Task Management API',
    'DESCRIPTION': 'Comprehensive task management system with real-time updates',
    'VERSION': '2.0.0',
    'SERVE_INCLUDE_SCHEMA': False,
    
    # API info
    'CONTACT': {
        'name': 'Deepcode Academy',
        'email': 'info@deepcode.academy',
        'url': 'https://deepcode.academy',
    },
    'LICENSE': {
        'name': 'MIT License',
    },
    
    # Swagger UI settings
    'SWAGGER_UI_SETTINGS': {
        'deepLinking': True,
        'persistAuthorization': True,
        'displayOperationId': True,
    },
    
    # Component split
    'COMPONENT_SPLIT_REQUEST': True,
    
    # Schema extensions
    'EXTENSIONS_INFO': {},
}
```

### 2.2 URLs

`myproject/urls.py`:

```python
from drf_spectacular.views import (
    SpectacularAPIView,
    SpectacularSwaggerView,
    SpectacularRedocView,
)

urlpatterns = [
    # OpenAPI schema
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    
    # Swagger UI
    path('api/docs/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
    
    # ReDoc
    path('api/redoc/', SpectacularRedocView.as_view(url_name='schema'), name='redoc'),
]
```

### 2.3 Schema Annotations

`tasks/views.py`:

```python
from drf_spectacular.utils import (
    extend_schema,
    extend_schema_view,
    OpenApiParameter,
    OpenApiExample,
    OpenApiResponse,
)
from drf_spectacular.types import OpenApiTypes

@extend_schema_view(
    list=extend_schema(
        summary='List all tasks',
        description='Get paginated list of tasks with filtering and search',
        parameters=[
            OpenApiParameter(
                name='status',
                type=OpenApiTypes.STR,
                location=OpenApiParameter.QUERY,
                description='Filter by status (todo, in_progress, done)',
                enum=['todo', 'in_progress', 'done'],
            ),
            OpenApiParameter(
                name='priority',
                type=OpenApiTypes.STR,
                description='Filter by priority',
                enum=['low', 'medium', 'high'],
            ),
            OpenApiParameter(
                name='search',
                type=OpenApiTypes.STR,
                description='Search in title and description',
            ),
        ],
        examples=[
            OpenApiExample(
                'List all tasks',
                value={'results': [{'id': 1, 'title': 'Task 1'}]},
                response_only=True,
            ),
        ],
    ),
    create=extend_schema(
        summary='Create a task',
        description='Create a new task for authenticated user',
        request=TaskSerializer,
        responses={
            201: TaskSerializer,
            400: OpenApiResponse(description='Bad request'),
        },
        examples=[
            OpenApiExample(
                'Create task example',
                value={
                    'title': 'Complete DRF course',
                    'description': 'Finish all 19 lessons',
                    'priority': 'high',
                    'due_date': '2024-12-31'
                },
                request_only=True,
            ),
        ],
    ),
)
class TaskViewSet(viewsets.ModelViewSet):
    """
    ViewSet for managing tasks
    
    Provides CRUD operations for tasks with:
    - Filtering by status, priority
    - Search by title, description
    - Pagination
    - Permission: IsAuthenticated
    """
    queryset = Task.objects.all()
    serializer_class = TaskSerializer
```

### 2.4 Serializer Documentation

`tasks/serializers.py`:

```python
from drf_spectacular.utils import extend_schema_field
from drf_spectacular.types import OpenApiTypes

class TaskSerializer(serializers.ModelSerializer):
    """
    Task serializer with all fields
    """
    
    owner = serializers.ReadOnlyField(source='owner.username')
    
    @extend_schema_field(OpenApiTypes.STR)
    def get_status_display(self, obj):
        """Human-readable status"""
        return obj.get_status_display()
    
    class Meta:
        model = Task
        fields = [
            'id', 'title', 'description', 'status', 
            'priority', 'due_date', 'owner', 'created_at'
        ]
        read_only_fields = ['id', 'owner', 'created_at']
```

---

## 🔒 3. CORS CONFIGURATION

### 3.1 Setup

```bash
pip install django-cors-headers
```

`myproject/settings.py`:

```python
INSTALLED_APPS = [
    # ...
    'corsheaders',
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',  # Before CommonMiddleware
    'django.middleware.common.CommonMiddleware',
    # ...
]

# CORS settings
# Development (allow all)
if DEBUG:
    CORS_ALLOW_ALL_ORIGINS = True
else:
    # Production (specific origins)
    CORS_ALLOWED_ORIGINS = [
        'https://example.com',
        'https://app.example.com',
    ]

# Allow credentials (cookies)
CORS_ALLOW_CREDENTIALS = True

# Allowed methods
CORS_ALLOW_METHODS = [
    'DELETE',
    'GET',
    'OPTIONS',
    'PATCH',
    'POST',
    'PUT',
]

# Allowed headers
CORS_ALLOW_HEADERS = [
    'accept',
    'accept-encoding',
    'authorization',
    'content-type',
    'dnt',
    'origin',
    'user-agent',
    'x-csrftoken',
    'x-requested-with',
]

# Preflight cache (seconds)
CORS_PREFLIGHT_MAX_AGE = 86400  # 24 hours
```

---

## ⚠️ 4. CUSTOM EXCEPTION HANDLING

### 4.1 Global Exception Handler

`tasks/exceptions.py`:

```python
from rest_framework.views import exception_handler
from rest_framework.response import Response
from rest_framework import status
import logging

logger = logging.getLogger(__name__)

def custom_exception_handler(exc, context):
    """
    Custom exception handler for consistent error responses
    
    Standard format:
    {
        "error": {
            "code": "error_code",
            "message": "Human-readable message",
            "details": {...},
            "timestamp": "2024-01-22T10:00:00Z"
        }
    }
    """
    # Call DRF's default handler first
    response = exception_handler(exc, context)
    
    if response is not None:
        # Log error
        logger.error(
            f'API Error: {exc.__class__.__name__} - {str(exc)}',
            exc_info=True,
            extra={'context': context}
        )
        
        # Customize response
        from django.utils import timezone
        
        custom_response = {
            'error': {
                'code': exc.__class__.__name__,
                'message': str(exc),
                'details': response.data,
                'timestamp': timezone.now().isoformat(),
                'path': context['request'].path,
            }
        }
        
        response.data = custom_response
    else:
        # Unhandled exception
        logger.critical(
            f'Unhandled Exception: {exc.__class__.__name__}',
            exc_info=True,
            extra={'context': context}
        )
    
    return response
```

### 4.2 Custom Exception Classes

```python
from rest_framework.exceptions import APIException

class TaskNotFoundError(APIException):
    """Custom exception for task not found"""
    status_code = 404
    default_detail = 'Task not found'
    default_code = 'task_not_found'


class TaskAlreadyCompletedError(APIException):
    """Custom exception for already completed task"""
    status_code = 400
    default_detail = 'Task is already completed'
    default_code = 'task_already_completed'


class InsufficientPermissionsError(APIException):
    """Custom exception for permission denied"""
    status_code = 403
    default_detail = 'You do not have permission to perform this action'
    default_code = 'insufficient_permissions'


# Usage in views
class TaskViewSet(viewsets.ModelViewSet):
    def complete(self, request, pk=None):
        task = self.get_object()
        
        if task.completed:
            raise TaskAlreadyCompletedError()
        
        task.completed = True
        task.save()
        
        return Response({'status': 'success'})
```

---

## 🛡️ 5. SECURITY BEST PRACTICES

### 5.1 Production Security Checklist

`myproject/settings_prod.py`:

```python
# SECURITY SETTINGS

# HTTPS/SSL
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')

# HSTS (HTTP Strict Transport Security)
SECURE_HSTS_SECONDS = 31536000  # 1 year
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# Content Security
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_BROWSER_XSS_FILTER = True
X_FRAME_OPTIONS = 'DENY'

# CSRF
CSRF_COOKIE_HTTPONLY = True
CSRF_USE_SESSIONS = True

# Session
SESSION_COOKIE_AGE = 86400  # 24 hours
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SAMESITE = 'Strict'

# Allowed hosts
ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# Secret key from environment
SECRET_KEY = os.environ.get('SECRET_KEY')
if not SECRET_KEY:
    raise ValueError('SECRET_KEY environment variable is not set')

# Debug off in production
DEBUG = False

# Admin URL obfuscation
ADMIN_URL = os.environ.get('ADMIN_URL', 'admin/')
```

### 5.2 Rate Limiting Strategies

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle',
        'rest_framework.throttling.ScopedRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/hour',
        'user': '1000/hour',
        'auth': '10/minute',  # Login attempts
        'api': '5000/hour',   # General API
    }
}
```

---

## 📊 6. MONITORING & LOGGING

### 6.1 Comprehensive Logging

`myproject/settings.py`:

```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '[{levelname}] {asctime} {name} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
        'simple': {
            'format': '[{levelname}] {message}',
            'style': '{',
        },
        'json': {
            '()': 'pythonjsonlogger.jsonlogger.JsonFormatter',
            'format': '%(asctime)s %(name)s %(levelname)s %(message)s',
        },
    },
    'filters': {
        'require_debug_false': {
            '()': 'django.utils.log.RequireDebugFalse',
        },
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'handlers': {
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
        'file': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': '/app/logs/django.log',
            'maxBytes': 1024 * 1024 * 10,  # 10MB
            'backupCount': 5,
            'formatter': 'verbose',
        },
        'error_file': {
            'level': 'ERROR',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': '/app/logs/django_errors.log',
            'maxBytes': 1024 * 1024 * 10,
            'backupCount': 5,
            'formatter': 'verbose',
        },
        'mail_admins': {
            'level': 'ERROR',
            'class': 'django.utils.log.AdminEmailHandler',
            'filters': ['require_debug_false'],
        },
    },
    'loggers': {
        'django': {
            'handlers': ['console', 'file', 'error_file'],
            'level': 'INFO',
        },
        'django.request': {
            'handlers': ['error_file', 'mail_admins'],
            'level': 'ERROR',
            'propagate': False,
        },
        'tasks': {
            'handlers': ['console', 'file'],
            'level': 'DEBUG',
        },
    },
}
```

### 6.2 Sentry Integration

```bash
pip install sentry-sdk
```

```python
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration
from sentry_sdk.integrations.celery import CeleryIntegration

sentry_sdk.init(
    dsn=os.environ.get('SENTRY_DSN'),
    integrations=[
        DjangoIntegration(),
        CeleryIntegration(),
    ],
    traces_sample_rate=1.0 if DEBUG else 0.1,
    send_default_pii=False,
    environment=os.environ.get('ENVIRONMENT', 'production'),
)
```

---

## 🎯 AMALIYOT TOPSHIRIQLARI

### 📝 Topshiriq 1: API Documentation (Oson)

**Talablar:**
- ✅ drf-spectacular setup
- ✅ Schema annotations (@extend_schema)
- ✅ Swagger UI
- ✅ API versioning (v1, v2)

### 📝 Topshiriq 2: Security Hardening (O'rta)

**Talablar:**
- ✅ CORS configuration
- ✅ Custom exception handler
- ✅ Rate limiting per endpoint
- ✅ Security headers
- ✅ Environment variables

### 📝 Topshiriq 3: Production-Ready API (Qiyin)

**Talablar:**
- ✅ Complete documentation
- ✅ Sentry integration
- ✅ Comprehensive logging
- ✅ Health check endpoint
- ✅ API versioning with deprecation
- ✅ 95%+ test coverage
- ✅ Performance monitoring

---

## 🔗 KURSNI YAKUNLASH

✅ **🎉 Tabriklaymiz! Siz Django REST Framework kursini tugatdingiz!**

**Siz o'rgandingiz:**
- ✅ RESTful API basics
- ✅ Authentication & Permissions
- ✅ Filtering, Pagination, Throttling
- ✅ JWT, Signals, Celery
- ✅ Testing, Caching, WebSockets
- ✅ Docker deployment
- ✅ Advanced concepts

**Keyingi qadamlar:**
1. O'z loyihangizni yarating
2. Portfolio uchun API publish qiling
3. Open source contribute qiling
4. Advanced topics: GraphQL, gRPC

---

## 📚 QISQA XULOSALAR

### API Best Practices

```python
# ✅ Must Have
- API versioning
- Auto-generated docs
- Custom exception handling
- CORS configuration
- Rate limiting
- Security headers
- Comprehensive logging
- Error monitoring (Sentry)
- Health checks
- Testing (95%+ coverage)

# ✅ Performance
- Database query optimization
- Caching strategy
- Async tasks (Celery)
- Connection pooling
- CDN for static files

# ✅ Code Quality
- Type hints
- Docstrings
- Clean code principles
- DRY (Don't Repeat Yourself)
- Separation of concerns
```

### Professional API Checklist

```python
# Documentation
□ Swagger/OpenAPI docs
□ README with examples
□ Postman collection
□ Changelog/Release notes

# Security
□ HTTPS only
□ JWT authentication
□ Permission system
□ Rate limiting
□ Input validation
□ SQL injection protection
□ XSS protection

# Monitoring
□ Error tracking (Sentry)
□ Performance monitoring
□ Log aggregation
□ Uptime monitoring
□ Alert system

# Testing
□ Unit tests
□ Integration tests
□ API tests
□ Load tests
□ Security tests
```

**Esda tuting:**
- Documentation = Developer happiness
- Security = Trust
- Monitoring = Peace of mind
- Testing = Confidence
- Keep learning! 🚀

---

**© 2024 Deepcode Academy. Barcha huquqlar himoyalangan.**

🌐 Website: [deepcode.academy](https://deepcode.academy)  
📱 Telegram: [@deepcode_academy](https://t.me/deepcode_academy)
