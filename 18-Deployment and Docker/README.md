# 🧩 18-DARS: JOYLASHTIRISH VA DOCKER BILAN ISHLASH

Bu darsda Django REST Framework (DRF) loyihasini Docker yordamida joylashtirishni bosqichma-bosqich o'rganamiz. Docker loyihani izolyatsiya qilingan konteynerlarda ishlatish imkonini beradi, bu esa uni turli muhitlarda barqaror ishlashini ta'minlaydi. Har bir qadam tushunarli va faqat joylashtirish va Docker ga qaratilgan bo'ladi. Oldingi darslarda sozlangan `myproject` loyihasi va `myapp` ilovasidagi `Task` modeli, JWT autentifikatsiyasi, ruxsatlar, sahifalash, Celery, keshlash va WebSocket (Django Channels) asosida davom etamiz.

## ✅ 1. TAYYORLOV ISHLARI
📌 Loyiha va ilova allaqachon sozlangan deb hisoblaymiz (`myproject`, `myapp`, va `Task` modeli). Quyidagi sozlamalar mavjud bo'lishi kerak:
- `Task` modeli `myapp/models.py` faylida belgilangan (shu jumladan `owner` maydoni).
- `TaskSerializer` `myapp/serializers.py` faylida yaratilgan.
- `TaskViewSet` `myapp/views.py` faylida sozlangan (JWT autentifikatsiyasi, ruxsatlar, filtrlash, sahifalash, keshlash va WebSocket bilan).
- Signallar, Celery va WebSocket `myapp/signals.py`, `myapp/tasks.py` va `myapp/consumers.py` fayllarida sozlangan.
- `/tasks/` va `/ws/tasks/` endpointlari `myproject/urls.py` va `myapp/routing.py` fayllarida sozlangan.
📌 Agar bu sozlamalar hali amalga oshirilmagan bo'lsa, avvalgi darslarga qayting.

## ✅ 2. DOCKER VA JOYLASHTIRISH NI TUSHUNISH
📌 **Docker**: Ilovalarni konteynerlarga joylashtirib, ularni mustaqil va moslashuvchan muhitda ishlatish imkonini beradi.
📌 **Joylashtirish**: Loyihani ishlab chiqarish (production) muhitida foydalanuvchilarga ochiq qilish jarayoni.
📌 Biz quyidagi komponentlarni Docker konteynerlarida joylashtiramiz:
- Django (Daphne bilan WebSocket uchun).
- Redis (Celery, keshlash va Channels uchun).
- PostgreSQL (ma'lumotlar bazasi sifatida).
- Celery worker.
📌 Nginx ni reverse proxy sifatida ishlatamiz.

## ✅ 3. DOCKER VA DOCKER COMPOSE NI O'RNATISH
📌 Docker va Docker Compose ni o'rnatish:
- **Docker**: [Rasmiy saytdan](https://docs.docker.com/get-docker/) o'rnatiladi.
- **Docker Compose**: Docker bilan birga keladi yoki alohida o'rnatiladi:
  ```bash
  sudo apt-get install docker-compose  # Ubuntu uchun
  ```
📌 Docker ishlayotganligini tekshiring:
```bash
docker --version
docker-compose --version
```

## ✅ 4. POSTGRESQL UCHUN SOZLAMALAR
📌 Loyiha SQLite dan PostgreSQL ga o'tkaziladi. `myproject/settings.py` faylini yangilaymiz:
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'rest_framework.authtoken',
    'django_filters',
    'channels',
    'myapp.apps.MyappConfig',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
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
    'PAGE_SIZE': 5,
}

# PostgreSQL sozlamalari
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydb',
        'USER': 'myuser',
        'PASSWORD': 'mypassword',
        'HOST': 'db',
        'PORT': '5432',
    }
}

# Celery sozlamalari
CELERY_BROKER_URL = 'redis://redis:6379/0'
CELERY_RESULT_BACKEND = 'redis://redis:6379/0'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_TIMEZONE = 'Asia/Tashkent'
CELERY_TASK_ALWAYS_EAGER = False  # Ishlab chiqarishda sinxron emas

# Keshlash sozlamalari
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://redis:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}

# Channels sozlamalari
ASGI_APPLICATION = 'myproject.asgi.application'
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            'hosts': [('redis', 6379)],
        },
    },
}
```
📌 **Tushuntirish**:
- `DATABASES`: PostgreSQL sozlamalari qo'shildi. `db` — Docker Compose dagi PostgreSQL xizmati nomi.
- `CELERY_BROKER_URL` va `CACHES`: `redis` xizmati nomiga o'zgartirildi.
- `CELERY_TASK_ALWAYS_EAGER = False`: Ishlab chiqarishda Celery asinxron ishlaydi.

## ✅ 5. DOCKERFILE YARATISH
📌 Loyiha ildizida `Dockerfile` yarating:
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

ENV PYTHONUNBUFFERED=1

CMD ["daphne", "-b", "0.0.0.0", "-p", "8000", "myproject.asgi:application"]
```
📌 **Tushuntirish**:
- `python:3.9-slim`: Yengil Python tasviri.
- `requirements.txt`: Loyiha bog'liqliklarini o'rnatadi.
- `daphne`: WebSocket uchun ASGI serveri sifatida ishlaydi.

## ✅ 6. REQUIREMENTS.TXT YARATISH
📌 Loyiha ildizida `requirements.txt` faylini yarating:
```
django==4.2
djangorestframework==3.14
django-filter==23.2
djangorestframework-simplejwt==5.2
channels==4.0
channels-redis==4.0
django-redis==5.2
celery==5.2
redis==4.5
psycopg2-binary==2.9
daphne==4.0
```
📌 **Eslatma**: Versiyalar sizning loyihangizga mos kelishi kerak.

## ✅ 7. DOCKER COMPOSE SOZLASH
📌 Loyiha ildizida `docker-compose.yml` faylini yarating:
```yaml
version: '3.8'

services:
  db:
    image: postgres:13
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:6
    volumes:
      - redis_data:/data

  web:
    build: .
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis
    environment:
      - PYTHONUNBUFFERED=1
    command: >
      sh -c "python manage.py migrate &&
             daphne -b 0.0.0.0 -p 8000 myproject.asgi:application"

  celery:
    build: .
    volumes:
      - .:/app
    depends_on:
      - redis
      - db
    command: celery -A myproject worker --loglevel=info

  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - web

volumes:
  postgres_data:
  redis_data:
```
📌 **Tushuntirish**:
- `db`: PostgreSQL xizmati.
- `redis`: Redis xizmati.
- `web`: Django va Daphne xizmati.
- `celery`: Celery worker xizmati.
- `nginx`: Reverse proxy sifatida ishlaydi.

## ✅ 8. NGINX SOZLASH
📌 Loyiha ildizida `nginx.conf` faylini yarating:
```nginx
events {}

http {
    server {
        listen 80;

        location / {
            proxy_pass http://web:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        location /ws/ {
            proxy_pass http://web:8000;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
        }
    }
}
```
📌 **Tushuntirish**:
- `/`: HTTP so'rovlarini `web` xizmatiga yo'naltiradi.
- `/ws/`: WebSocket so'rovlarini qo'llab-quvvatlaydi.

## ✅ 9. LOYIHANI JOYLASHTIRISH
📌 Docker Compose bilan loyihani ishga tushirish:
```bash
docker-compose up --build
```
📌 Migratsiyalarni amalga oshirish (birinchi marta):
```bash
docker-compose exec web python manage.py migrate
```
📌 Superuser yaratish:
```bash
docker-compose exec web python manage.py createsuperuser
```

## ✅ 10. JOYLASHTIRISHNI SINAB KO'RISH
📌 Brauzerda `http://localhost` yoki `http://localhost/ws/` manzilini oching.
📌 API ni sinash:
```bash
curl -X POST http://localhost/api/token/ -H "Content-Type: application/json" -d '{"username": "testuser", "password": "testpass123"}'
```
```bash
curl -X POST http://localhost/tasks/ -H "Authorization: Bearer <access_token>" -H "Content-Type: application/json" -d '{"title": "Yangi vazifa", "description": "Test"}'
```
📌 WebSocket sinovi:
- `http://localhost/ws/` manzilida WebSocket sahifasini oching.
- Yangi vazifa qo'shing va bildirishnoma oling.

## ✅ 11. TESTLARNI DOCKERDA ISHGA TUSHIRISH
📌 Testlarni Docker ichida ishga tushirish:
```bash
docker-compose exec web python manage.py test
```

## ✅ 12. SINOV UCHUN MASALALAR
📌 Joylashtirishni sinash uchun quyidagi amallarni bajarib ko'ring:
1. `docker-compose up --build` bilan loyihani ishga tushiring va `http://localhost/tasks/` manzilini sinang.
2. WebSocket ni `http://localhost/ws/` manzilida sinab, vazifa qo'shishda bildirishnoma oling.
3. Testlarni `docker-compose exec web python manage.py test` bilan ishga tushiring.
4. PostgreSQL ga ulanishni `docker-compose exec db psql -U myuser -d mydb` bilan tekshiring.

