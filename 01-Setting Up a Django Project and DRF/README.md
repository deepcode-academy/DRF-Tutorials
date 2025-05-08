# Setting Up a Django Project and DRF

# 📌 Set up the environment (Virtual Environment)

## 📌 Create a virtual environment:

```shell
python -m venv env
```

## 📌 Activate the virtual environment:
### Windows

```shell
env\Scripts\activate
```

### Linux/macOS:

```shell
source venv/bin/activate
```

## 📌 Install Django and DRF:

```shell
pip install django
```

```shell
pip install djangorestframework
```

# 📁 Create Django Project and App

## 📌 Create Django project:

```shell
django-admin startproject config .
cd config
```

## 📌 Create Django app:

```shell
python manage.py startapp api
```

# ⚙️ Configure settings.py

## 📌 Add apps to INSTALLED_APPS:

```python
INSTALLED_APPS = [
    # ...
    'rest_framework',
    'api',
]
```

