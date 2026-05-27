# ╔══════════════════════════════════════════════════════════════╗
# ║  COMPLETE PYTHON BACKEND INTERVIEW GUIDE                    ║
# ║  Django • FastAPI • Flask                                   ║
# ║  Beginner → Intermediate → Advanced → Expert               ║
# ║  Interview-Ready | Simple Language | Production Examples    ║
# ╚══════════════════════════════════════════════════════════════╝

> Quick-revise this before any interview. Every answer is short, clear, and interview-ready.
> Simple explanations first → technical depth → code example → common mistakes.

---

# TABLE OF CONTENTS

- PART 1: DJANGO (Complete)
- PART 2: FASTAPI (Complete)
- PART 3: FLASK (Complete)
- PART 4: COMPARISON TABLES
- PART 5: SECURITY DEEP DIVE
- PART 6: DEPLOYMENT & DEVOPS
- PART 7: DATABASE
- PART 8: TOP INTERVIEW QUESTIONS (Django 200 + FastAPI 150 + Flask 100)
- PART 9: CODING EXAMPLES
- PART 10: SYSTEM DESIGN BASICS

---

# ══════════════════════════════════════════
# PART 1: DJANGO — COMPLETE GUIDE
# ══════════════════════════════════════════

---

## 1.1 WHAT IS DJANGO?

**Simple Answer:**
Django is a high-level Python web framework that lets you build web apps quickly.
Think of it as a fully-stocked kitchen — oven (ORM), plates (templates), chef tools (forms, auth, admin) — everything included.

**Why Django?**
- "Batteries included" — ORM, Auth, Admin, Forms, Caching all built-in
- Rapid development
- Secure by default (CSRF, XSS, SQL injection protection)
- Used by: Instagram, Pinterest, Disqus, Mozilla, NASA

**Interview Answer:**
> "Django is a Python web framework that follows the MTV pattern. It includes an ORM, admin panel, authentication, and security features out of the box. It helps developers build production-ready applications quickly."

---

## 1.2 DJANGO MTV ARCHITECTURE

**Simple Explanation:**
MTV = Model + Template + View (Django's version of MVC)

```
MVC  →  MTV
Model    → Model     (database layer)
View     → Template  (what user sees)
Controller → View    (logic layer)
```

**Flow:**
```
Browser → URL Router → View → Model (DB) → View → Template → Browser
```

**Interview Answer:**
> "Django uses MTV. Model handles data/DB. View handles business logic. Template handles presentation. URLs route requests to the right View."

---

## 1.3 DJANGO REQUEST-RESPONSE CYCLE

```
HTTP Request
    ↓
WSGI Server (Gunicorn)
    ↓
Django Middleware Stack ← top to bottom
    ↓
URL Router (urls.py)
    ↓
View (function or class)
    ↓
Model (ORM → Database)  ←  Cache Check
    ↓
Response Object
    ↓
Middleware Stack ← bottom to top
    ↓
HTTP Response → Browser
```

**Key points:**
- Middleware runs on EVERY request and response
- URL router matches pattern → calls view
- View can query DB, render template, or return JSON

---

## 1.4 WSGI vs ASGI

| Feature     | WSGI                        | ASGI                         |
|-------------|-----------------------------|------------------------------|
| Stands for  | Web Server Gateway Interface | Async Server Gateway Interface |
| Type        | Synchronous                 | Asynchronous                 |
| Handles     | HTTP only                   | HTTP + WebSocket + HTTP/2    |
| Server      | Gunicorn                    | Uvicorn, Daphne              |
| Django use  | Default (traditional)       | Django 3.1+ (Channels)       |
| Concurrency | Multiple processes           | Async coroutines             |

**Simple Explanation:**
- WSGI = Old phone — one call at a time
- ASGI = Modern phone — multiple calls, WhatsApp, emails simultaneously

---

## 1.5 DJANGO PROJECT STRUCTURE

```
myproject/
├── manage.py              ← CLI tool (runserver, migrate, etc.)
├── config/
│   ├── settings.py        ← All configurations
│   ├── urls.py            ← Root URL routing
│   ├── wsgi.py            ← WSGI entry point
│   └── asgi.py            ← ASGI entry point
└── apps/
    └── users/
        ├── models.py      ← Database models
        ├── views.py       ← Business logic
        ├── urls.py        ← App URL routing
        ├── serializers.py ← DRF serializers
        ├── admin.py       ← Admin registration
        ├── signals.py     ← Event handlers
        └── tests.py       ← Tests
```

---

## 1.6 DJANGO MODELS & ORM

**Simple Explanation:**
Model = blueprint for your database table.
ORM = translator — converts Python code to SQL automatically.

```python
# models.py
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-created_at']
        indexes = [models.Index(fields=['name', 'is_active'])]
    
    def __str__(self):
        return self.name

# ORM Queries
Product.objects.all()                          # SELECT * FROM products
Product.objects.filter(is_active=True)         # WHERE is_active = TRUE
Product.objects.get(id=1)                      # Get single or raise exception
Product.objects.create(name="Laptop", price=50000)
Product.objects.filter(id=1).update(price=55000)
Product.objects.filter(id=1).delete()
Product.objects.filter(price__gt=1000)         # price > 1000
Product.objects.filter(name__icontains="lap")  # case-insensitive contains
Product.objects.order_by('-price')[:10]        # ORDER BY price DESC LIMIT 10
```

**Field Types (must know):**

| Field               | Use For                     |
|---------------------|-----------------------------|
| CharField           | Short text, max_length req  |
| TextField           | Long text, no max_length    |
| IntegerField        | Integer numbers             |
| DecimalField        | Money/precise decimals      |
| BooleanField        | True/False                  |
| DateTimeField       | Date + time                 |
| ForeignKey          | Many-to-one relationship    |
| ManyToManyField     | Many-to-many relationship   |
| OneToOneField       | One-to-one relationship     |
| EmailField          | Email with validation       |
| ImageField/FileField| File uploads                |
| JSONField           | JSON data (PostgreSQL)      |

---

## 1.7 QuerySet — LAZY EVALUATION

**Simple Explanation:**
QuerySet doesn't hit the DB until you actually need the data. Like ordering food online — you build the order (queryset), but food is only prepared (DB query) when you pay (evaluate).

```python
# Building queryset — NO DB QUERY YET
qs = Product.objects.filter(is_active=True)
qs = qs.filter(price__lt=5000)
qs = qs.order_by('name')

# DB QUERY happens here (evaluation)
list(qs)           # ← Query executes
for p in qs:       # ← Query executes
qs[0]              # ← Query executes
qs.count()         # ← Query executes (SELECT COUNT)
qs.exists()        # ← Query executes (SELECT 1... LIMIT 1)
```

---

## 1.8 select_related vs prefetch_related

**⭐ Most asked Django interview question!**

**Simple Explanation:**
- `select_related` = JOINS in one SQL query (for FK/OneToOne → "one" side)
- `prefetch_related` = Two queries, Python joins (for M2M/reverse FK → "many" side)

```python
# Models
class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)

class Tag(models.Model):
    name = models.CharField(max_length=50)
    books = models.ManyToManyField(Book)

# ❌ N+1 Problem — 1 + N queries!
books = Book.objects.all()        # 1 query
for b in books:
    print(b.author.name)          # 1 query per book = N extra queries!

# ✅ select_related (ForeignKey) — Single JOIN query
books = Book.objects.select_related('author').all()  # 1 query total

# ✅ prefetch_related (ManyToMany) — 2 queries total
books = Book.objects.prefetch_related('tag_set').all()

# Chain them
books = Book.objects.select_related('author').prefetch_related('tag_set')
```

**Interview Answer:**
> "select_related uses SQL JOIN for FK/OneToOne — one query. prefetch_related makes 2 queries and joins in Python — for M2M or reverse FKs. Both solve the N+1 problem."

---

## 1.9 MIGRATIONS

**Simple Explanation:**
Migrations = version control for your database schema.

```bash
python manage.py makemigrations      # Detect model changes → create migration file
python manage.py migrate             # Apply migrations to DB
python manage.py showmigrations      # Show all migration status
python manage.py sqlmigrate app 0001 # Show SQL for a migration
python manage.py migrate app 0001    # Rollback to specific migration
python manage.py migrate app zero    # Undo all migrations for an app
```

**Common mistakes:**
- Deleting migration files without running `migrate zero` first
- Conflicting migrations in team — use `makemigrations --merge`
- Forgetting to add migrations to git

---

## 1.10 CUSTOM MANAGERS

```python
class ActiveManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(is_active=True)

class Product(models.Model):
    name = models.CharField(max_length=200)
    is_active = models.BooleanField(default=True)
    
    objects = models.Manager()        # Default manager
    active = ActiveManager()          # Custom manager

# Usage
Product.active.all()                  # Only active products
Product.objects.all()                 # All products
```

---

## 1.11 MIDDLEWARE

**Simple Explanation:**
Middleware = security checkpoint. Every request passes through it going IN, and every response passes through it going OUT.

```python
# custom middleware
import time
import uuid

class RequestTimingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response  # Called once on startup
    
    def __call__(self, request):
        # BEFORE view
        request.correlation_id = str(uuid.uuid4())
        start = time.perf_counter()
        
        response = self.get_response(request)  # View runs here
        
        # AFTER view
        elapsed = time.perf_counter() - start
        response['X-Response-Time'] = str(elapsed)
        return response

# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'myapp.middleware.RequestTimingMiddleware',  # Add yours
    ...
]
```

**Built-in Django Middleware (know these):**
- `SecurityMiddleware` — HTTPS redirect, security headers
- `SessionMiddleware` — Enables sessions
- `CsrfViewMiddleware` — CSRF protection
- `AuthenticationMiddleware` — Attaches user to request
- `MessageMiddleware` — Flash messages

---

## 1.12 SIGNALS

**Simple Explanation:**
Signals = event notifications. Like WhatsApp groups — when something happens (signal), all subscribers are notified automatically.

```python
from django.db.models.signals import post_save, pre_save, post_delete
from django.dispatch import receiver

# Auto-create profile when user is created
@receiver(post_save, sender=User)
def create_profile(sender, instance, created, **kwargs):
    if created:
        UserProfile.objects.create(user=instance)
        send_welcome_email.delay(instance.email)  # Celery task

# Log before deletion
@receiver(pre_save, sender=Order)
def log_status_change(sender, instance, **kwargs):
    if instance.pk:  # Existing object
        old = Order.objects.get(pk=instance.pk)
        if old.status != instance.status:
            OrderLog.objects.create(order=instance, change=f"{old.status}→{instance.status}")

# Register signals — in app's ready()
# apps.py
class UsersConfig(AppConfig):
    def ready(self):
        import users.signals  # Import to connect signals
```

**Signals vs Middleware:**

| Signals                  | Middleware               |
|--------------------------|--------------------------|
| Model-level events       | Request/Response level   |
| post_save, pre_delete    | Every HTTP request       |
| Good for side effects    | Good for cross-cutting   |
| Hard to trace            | Easy to trace            |

---

## 1.13 DJANGO VIEWS

### Function-Based Views (FBV)
```python
from django.http import JsonResponse
from django.views.decorators.http import require_http_methods
from django.contrib.auth.decorators import login_required

@login_required
@require_http_methods(["GET", "POST"])
def user_list(request):
    if request.method == "GET":
        users = User.objects.all().values('id', 'email', 'name')
        return JsonResponse(list(users), safe=False)
    
    if request.method == "POST":
        data = json.loads(request.body)
        user = User.objects.create(**data)
        return JsonResponse({"id": user.id}, status=201)
```

### Class-Based Views (CBV)
```python
from django.views import View
from django.contrib.auth.mixins import LoginRequiredMixin

class UserListView(LoginRequiredMixin, View):
    def get(self, request):
        users = User.objects.all()
        return JsonResponse({"users": list(users.values())})
    
    def post(self, request):
        data = json.loads(request.body)
        user = User.objects.create(**data)
        return JsonResponse({"id": user.id}, status=201)
```

**FBV vs CBV:**

| FBV                        | CBV                         |
|----------------------------|-----------------------------|
| Simpler to understand      | More reusable via inheritance|
| Less boilerplate for simple | Less code for complex CRUD  |
| Less flexible for extension| Mixins for shared behavior  |
| Good for: simple views     | Good for: CRUD, reusability |

---

## 1.14 DJANGO REST FRAMEWORK (DRF)

### View Hierarchy
```
APIView         → Full control, write everything
  └── GenericAPIView → adds queryset, serializer_class
       └── Mixins → ListModelMixin, CreateModelMixin, etc.
            └── Generic Views → ListCreateAPIView, etc.
                 └── ViewSet → cleaner, router-compatible
                      └── ModelViewSet → CRUD for FREE
```

### APIView
```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status

class UserAPIView(APIView):
    permission_classes = [IsAuthenticated]
    
    def get(self, request):
        users = User.objects.all()
        serializer = UserSerializer(users, many=True)
        return Response(serializer.data)
    
    def post(self, request):
        serializer = UserSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

### ModelViewSet (most powerful)
```python
from rest_framework import viewsets
from rest_framework.decorators import action
from rest_framework.routers import DefaultRouter

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = [IsAuthenticated]
    
    def get_queryset(self):
        if self.request.user.is_staff:
            return User.objects.all()
        return User.objects.filter(is_active=True)
    
    def get_serializer_class(self):
        if self.action == 'create':
            return UserCreateSerializer
        return UserSerializer
    
    @action(detail=True, methods=['post'])
    def deactivate(self, request, pk=None):
        user = self.get_object()
        user.is_active = False
        user.save()
        return Response({"status": "deactivated"})

# urls.py — Router auto-generates URLs
router = DefaultRouter()
router.register('users', UserViewSet, basename='user')
urlpatterns = router.urls
# Creates: GET/POST /users/, GET/PUT/PATCH/DELETE /users/{id}/
# POST /users/{id}/deactivate/
```

### Serializers
```python
from rest_framework import serializers

class UserSerializer(serializers.ModelSerializer):
    full_name = serializers.SerializerMethodField()
    password = serializers.CharField(write_only=True)
    
    class Meta:
        model = User
        fields = ['id', 'email', 'password', 'full_name', 'created_at']
        read_only_fields = ['id', 'created_at']
    
    def get_full_name(self, obj):
        return f"{obj.first_name} {obj.last_name}"
    
    def validate_email(self, value):
        if User.objects.filter(email=value).exists():
            raise serializers.ValidationError("Email already exists")
        return value.lower()
    
    def create(self, validated_data):
        password = validated_data.pop('password')
        user = User(**validated_data)
        user.set_password(password)  # Hash password!
        user.save()
        return user
```

---

## 1.15 JWT AUTHENTICATION (Step by Step)

**Simple Explanation:**
JWT = passport. Contains your identity info. Server checks passport validity without asking the database every time.

```
Step 1: User logs in (email + password)
Step 2: Server verifies credentials
Step 3: Server creates JWT → sends to client
Step 4: Client stores JWT (localStorage/cookie)
Step 5: Client sends JWT in every request header
Step 6: Server decodes JWT → no DB lookup needed
Step 7: If expired → client uses Refresh Token to get new Access Token
```

**JWT Structure:**
```
header.payload.signature
eyJhbGci...  .eyJ1c2VyX2...  .SflKxwRJSM...

Header:  { "alg": "HS256", "typ": "JWT" }
Payload: { "sub": "123", "email": "user@mail.com", "exp": 1735000000 }
Signature: HMAC(base64(header) + "." + base64(payload), SECRET_KEY)
```

**Access Token vs Refresh Token:**

| Access Token       | Refresh Token        |
|--------------------|----------------------|
| Short-lived (15min)| Long-lived (7 days)  |
| Used for API calls | Used to get new AT   |
| Stored in memory   | Stored in httpOnly cookie |
| Sent every request | Sent only to /refresh|

```python
# settings.py with djangorestframework-simplejwt
from datetime import timedelta
SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'ALGORITHM': 'HS256',
}

# urls.py
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView
urlpatterns = [
    path('auth/login/', TokenObtainPairView.as_view()),      # POST → access + refresh tokens
    path('auth/refresh/', TokenRefreshView.as_view()),        # POST → new access token
]

# Usage in request header:
# Authorization: Bearer <access_token>
```

---

## 1.16 DRF PERMISSIONS

```python
from rest_framework.permissions import BasePermission, SAFE_METHODS

# Custom permission
class IsOwnerOrReadOnly(BasePermission):
    def has_object_permission(self, request, view, obj):
        if request.method in SAFE_METHODS:  # GET, HEAD, OPTIONS
            return True
        return obj.owner == request.user

class IsAdminUser(BasePermission):
    def has_permission(self, request, view):
        return request.user and request.user.is_staff

# Usage
class ProductViewSet(viewsets.ModelViewSet):
    def get_permissions(self):
        if self.action in ['list', 'retrieve']:
            return [AllowAny()]
        elif self.action in ['create']:
            return [IsAuthenticated()]
        return [IsAuthenticated(), IsOwnerOrReadOnly()]
```

---

## 1.17 DJANGO CACHING

```python
# settings.py
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",
        "OPTIONS": {"CLIENT_CLASS": "django_redis.client.DefaultClient"},
        "TIMEOUT": 300,
    }
}

# View-level cache
from django.views.decorators.cache import cache_page

@cache_page(60 * 15)  # 15 minutes
def product_list(request): ...

# Low-level cache
from django.core.cache import cache

def get_product(product_id):
    key = f"product:{product_id}"
    data = cache.get(key)
    if data is None:
        data = Product.objects.get(id=product_id)
        cache.set(key, data, 300)
    return data

# Invalidate cache on update
def update_product(product_id, data):
    Product.objects.filter(id=product_id).update(**data)
    cache.delete(f"product:{product_id}")
```

---

## 1.18 CELERY WITH DJANGO

```python
# celery.py
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')
app = Celery('myproject')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()

# settings.py
CELERY_BROKER_URL = 'redis://localhost:6379/0'
CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'

# tasks.py
from celery import shared_task

@shared_task(bind=True, max_retries=3, default_retry_delay=60)
def send_email_task(self, user_id: int):
    try:
        user = User.objects.get(id=user_id)
        send_email(user.email, "Welcome!")
        return {"status": "sent"}
    except Exception as exc:
        raise self.retry(exc=exc)

# Calling tasks
send_email_task.delay(user_id=123)                          # Immediate
send_email_task.apply_async(args=[123], countdown=300)      # After 5 min

# Run worker
# celery -A config worker --loglevel=info
# celery -A config beat --loglevel=info  (for scheduled tasks)
```

---

## 1.19 DJANGO CHANNELS (WebSockets)

```python
# consumers.py
from channels.generic.websocket import AsyncWebsocketConsumer
import json

class ChatConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_group_name = f"chat_{self.room_name}"
        
        await self.channel_layer.group_add(self.room_group_name, self.channel_name)
        await self.accept()
    
    async def disconnect(self, close_code):
        await self.channel_layer.group_discard(self.room_group_name, self.channel_name)
    
    async def receive(self, text_data):
        data = json.loads(text_data)
        await self.channel_layer.group_send(
            self.room_group_name,
            {"type": "chat_message", "message": data['message']}
        )
    
    async def chat_message(self, event):
        await self.send(text_data=json.dumps({"message": event["message"]}))
```

---

## 1.20 MODEL INHERITANCE

### Abstract Model (most common)
```python
class TimeStampedModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        abstract = True  # No table created for this!

class Product(TimeStampedModel):  # Gets created_at, updated_at
    name = models.CharField(max_length=200)
```

### Proxy Model
```python
class ActiveProductManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(is_active=True)

class ActiveProduct(Product):  # SAME table as Product
    objects = ActiveProductManager()
    class Meta:
        proxy = True
```

### Multi-table Inheritance
```python
class Animal(models.Model):
    name = models.CharField(max_length=100)

class Dog(Animal):  # Separate table, linked via OneToOne
    breed = models.CharField(max_length=100)

# Dog table has: id, breed, animal_ptr_id (FK to Animal)
```

---

## 1.21 CUSTOM USER MODEL

**Must do at project START. Very painful to change later!**

```python
# models.py
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    email = models.EmailField(unique=True)
    phone = models.CharField(max_length=15, blank=True)
    avatar = models.ImageField(upload_to='avatars/', null=True, blank=True)
    
    USERNAME_FIELD = 'email'   # Login with email
    REQUIRED_FIELDS = ['username']

# settings.py
AUTH_USER_MODEL = 'users.User'  # Must add before first migration!
```

---

## 1.22 DJANGO SECURITY

### CSRF
```python
# Django auto-protects forms with CSRF token
# For APIs (JWT), add:
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt  # Safe because JWT auth replaces CSRF
def api_view(request): ...

# Better: Use DRF which handles this automatically
```

### CORS
```python
# pip install django-cors-headers
INSTALLED_APPS = ['corsheaders', ...]
MIDDLEWARE = ['corsheaders.middleware.CorsMiddleware', ...]

CORS_ALLOWED_ORIGINS = ["https://myapp.com", "https://admin.myapp.com"]
CORS_ALLOW_CREDENTIALS = True
```

### SQL Injection Prevention
```python
# Safe — ORM parameterizes automatically
User.objects.filter(email=user_input)

# Safe — parameterized raw query
User.objects.raw("SELECT * FROM users WHERE email = %s", [user_input])

# DANGEROUS — never do this
User.objects.raw(f"SELECT * FROM users WHERE email = '{user_input}'")
```

### XSS Prevention
```python
# Django templates auto-escape HTML
{{ user_content }}          # Safe — HTML escaped
{{ user_content|safe }}     # DANGEROUS — turns off escaping

# DRF Response auto-escapes JSON
```

---

## 1.23 PAGINATION IN DRF

```python
from rest_framework.pagination import PageNumberPagination, CursorPagination

class StandardPagination(PageNumberPagination):
    page_size = 20
    page_size_query_param = 'page_size'
    max_page_size = 100
    
    def get_paginated_response(self, data):
        return Response({
            'count': self.page.paginator.count,
            'next': self.get_next_link(),
            'previous': self.get_previous_link(),
            'results': data
        })

# settings.py
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'common.pagination.StandardPagination',
    'PAGE_SIZE': 20,
}
```

---

## 1.24 FILTERING & SEARCHING

```python
# pip install django-filter
from django_filters import rest_framework as filters
from rest_framework.filters import SearchFilter, OrderingFilter

class ProductFilter(filters.FilterSet):
    min_price = filters.NumberFilter(field_name='price', lookup_expr='gte')
    max_price = filters.NumberFilter(field_name='price', lookup_expr='lte')
    
    class Meta:
        model = Product
        fields = ['category', 'is_active', 'min_price', 'max_price']

class ProductViewSet(viewsets.ModelViewSet):
    filter_backends = [filters.DjangoFilterBackend, SearchFilter, OrderingFilter]
    filterset_class = ProductFilter
    search_fields = ['name', 'description']   # ?search=laptop
    ordering_fields = ['price', 'created_at'] # ?ordering=-price
    ordering = ['-created_at']                # default ordering
```

---

## 1.25 DJANGO TESTING

```python
from django.test import TestCase, Client
from rest_framework.test import APITestCase, APIClient
from django.contrib.auth import get_user_model

User = get_user_model()

class ProductAPITest(APITestCase):
    def setUp(self):
        self.user = User.objects.create_user(email='test@test.com', password='pass123')
        self.client = APIClient()
        self.client.force_authenticate(user=self.user)
    
    def test_list_products(self):
        Product.objects.create(name="Laptop", price=50000)
        response = self.client.get('/api/products/')
        self.assertEqual(response.status_code, 200)
        self.assertEqual(len(response.data['results']), 1)
    
    def test_create_product_unauthorized(self):
        self.client.force_authenticate(user=None)
        response = self.client.post('/api/products/', {'name': 'Test', 'price': 100})
        self.assertEqual(response.status_code, 401)

# Run tests
# python manage.py test
# pytest --cov=apps --cov-report=html
```

---

## 1.26 PERFORMANCE OPTIMIZATION

```python
# 1. Use values() when you don't need model instances
User.objects.all().values('id', 'email')        # Returns dicts, faster
User.objects.all().values_list('email', flat=True)  # Returns flat list

# 2. Use only() / defer()
User.objects.only('id', 'email')    # Fetch only these fields
User.objects.defer('bio', 'photo')  # Fetch everything EXCEPT these

# 3. Bulk operations
User.objects.bulk_create([User(...), User(...)])        # 1 query for all
User.objects.filter(...).update(is_active=True)         # 1 query for all
User.objects.bulk_update(user_list, ['is_active'])      # 1 query for all

# 4. Database connection reuse
# settings.py
DATABASES = {'default': {'CONN_MAX_AGE': 60}}

# 5. Annotate instead of Python loops
from django.db.models import Count, Avg
Category.objects.annotate(product_count=Count('product'))

# 6. Use exists() instead of count() or bool()
if User.objects.filter(email=email).exists():  # Faster than .count() or bool(.all())
    raise ValidationError("Email exists")
```

---

# ══════════════════════════════════════════
# PART 2: FASTAPI — COMPLETE GUIDE
# ══════════════════════════════════════════

---

## 2.1 WHAT IS FASTAPI?

**Simple Answer:**
FastAPI is a modern Python web framework for building APIs. It's like Django but designed specifically for speed and async APIs.

**Why FastAPI is Fast:**
1. **ASGI-native** — async from the ground up
2. **Pydantic** — fast data validation using Rust
3. **No ORM overhead** — use raw async SQLAlchemy
4. **Type hints** — zero-overhead validation
5. **Starlette underneath** — production-grade ASGI toolkit

**Key advantages:**
- Auto-generates Swagger docs (`/docs`) and ReDoc (`/redoc`)
- Type hint-based validation — no separate validation layer
- Async support natively
- Comparable speed to Node.js and Go

---

## 2.2 FASTAPI REQUEST-RESPONSE LIFECYCLE

```
HTTP Request
    ↓
ASGI Server (Uvicorn)
    ↓
Middleware Stack (top → bottom)
    ↓
Exception Handlers
    ↓
Route Matching
    ↓
Dependencies (Depends) — run in order
    ↓
Endpoint Function (async/sync)
    ↓
Response Model validation
    ↓
Middleware Stack (bottom → top)
    ↓
HTTP Response
```

---

## 2.3 FASTAPI BASICS

```python
from fastapi import FastAPI, Path, Query, Body, Header, HTTPException, status, Depends
from pydantic import BaseModel, Field, EmailStr
from typing import Optional, List
import uvicorn

app = FastAPI(
    title="My API",
    version="1.0.0",
    description="Production FastAPI app",
    docs_url="/docs",
    redoc_url="/redoc",
)

# Path parameter — part of URL
@app.get("/users/{user_id}")
async def get_user(user_id: int = Path(..., ge=1, title="User ID")):
    return {"user_id": user_id}

# Query parameter — ?key=value in URL
@app.get("/products")
async def list_products(
    skip: int = Query(0, ge=0),
    limit: int = Query(20, ge=1, le=100),
    category: Optional[str] = None,
    in_stock: bool = True,
):
    return {"skip": skip, "limit": limit}

# Request body
class UserCreate(BaseModel):
    name: str = Field(..., min_length=2, max_length=100)
    email: EmailStr
    age: int = Field(..., ge=18)

@app.post("/users", status_code=status.HTTP_201_CREATED)
async def create_user(user: UserCreate):
    return {"id": 1, **user.dict()}

if __name__ == "__main__":
    uvicorn.run("main:app", host="0.0.0.0", port=8000, reload=True)
```

---

## 2.4 PYDANTIC v2

```python
from pydantic import (
    BaseModel, Field, field_validator, model_validator,
    computed_field, EmailStr, ConfigDict
)
from typing import Optional, List, Literal
from datetime import datetime
from decimal import Decimal

class OrderItem(BaseModel):
    product_id: int
    quantity: int = Field(..., ge=1)
    unit_price: Decimal = Field(..., gt=0)

class OrderCreate(BaseModel):
    model_config = ConfigDict(
        str_strip_whitespace=True,
        validate_assignment=True,
    )
    
    customer_email: EmailStr
    items: List[OrderItem] = Field(..., min_length=1)
    status: Literal["pending", "confirmed", "shipped"] = "pending"
    notes: Optional[str] = Field(None, max_length=500)
    
    @field_validator('customer_email')
    @classmethod
    def email_lowercase(cls, v):
        return v.lower()
    
    @model_validator(mode='after')
    def check_items_not_empty(self):
        if not self.items:
            raise ValueError("Order must have at least one item")
        return self
    
    @computed_field
    @property
    def total_amount(self) -> Decimal:
        return sum(item.unit_price * item.quantity for item in self.items)

# Pydantic for Settings
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    DATABASE_URL: str
    REDIS_URL: str = "redis://localhost:6379/0"
    SECRET_KEY: str
    DEBUG: bool = False
    ALLOWED_HOSTS: List[str] = ["*"]
    
    class Config:
        env_file = ".env"

settings = Settings()  # Auto-reads .env
```

---

## 2.5 DEPENDENCY INJECTION

**Simple Explanation:**
Dependencies = helpers that get auto-called before your endpoint. Like a helper who prepares everything before the chef starts cooking.

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy.ext.asyncio import AsyncSession
from jose import JWTError, jwt

# DB Session dependency
async def get_db():
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise

# Auth dependency
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/token")

async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: AsyncSession = Depends(get_db)
) -> User:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        user_id: int = payload.get("sub")
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")
    
    user = await db.get(User, user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

async def get_current_active_user(
    user: User = Depends(get_current_user)
) -> User:
    if not user.is_active:
        raise HTTPException(status_code=400, detail="Inactive user")
    return user

# Use in endpoint
@app.get("/me")
async def get_my_profile(
    current_user: User = Depends(get_current_active_user),
    db: AsyncSession = Depends(get_db)
):
    return current_user

# Class-based dependency
class PaginationParams:
    def __init__(
        self,
        page: int = Query(1, ge=1),
        page_size: int = Query(20, ge=1, le=100)
    ):
        self.offset = (page - 1) * page_size
        self.limit = page_size

@app.get("/products")
async def list_products(
    pagination: PaginationParams = Depends(),
    db: AsyncSession = Depends(get_db)
):
    return {"offset": pagination.offset, "limit": pagination.limit}
```

---

## 2.6 ASYNC SQLAlchemy + CRUD

```python
# db/session.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase

engine = create_async_engine(
    "postgresql+asyncpg://user:pass@localhost/db",
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True,
)

AsyncSessionLocal = async_sessionmaker(engine, expire_on_commit=False)

class Base(DeclarativeBase):
    pass

# models/user.py
from sqlalchemy import Column, Integer, String, Boolean
class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    hashed_password = Column(String)
    is_active = Column(Boolean, default=True)

# crud/user.py
from sqlalchemy import select

class UserCRUD:
    async def get(self, db: AsyncSession, id: int):
        return await db.get(User, id)
    
    async def get_by_email(self, db: AsyncSession, email: str):
        result = await db.execute(select(User).where(User.email == email))
        return result.scalar_one_or_none()
    
    async def get_multi(self, db: AsyncSession, skip=0, limit=100):
        result = await db.execute(select(User).offset(skip).limit(limit))
        return result.scalars().all()
    
    async def create(self, db: AsyncSession, obj_in: UserCreate):
        db_obj = User(**obj_in.model_dump())
        db.add(db_obj)
        await db.flush()
        await db.refresh(db_obj)
        return db_obj

crud_user = UserCRUD()
```

---

## 2.7 FASTAPI EXCEPTION HANDLING

```python
from fastapi import Request
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError

# Custom exceptions
class NotFoundException(Exception):
    def __init__(self, resource: str, id: int):
        self.resource = resource
        self.id = id

# Global handlers
@app.exception_handler(NotFoundException)
async def not_found_handler(request: Request, exc: NotFoundException):
    return JSONResponse(
        status_code=404,
        content={"error": "NOT_FOUND", "message": f"{exc.resource} {exc.id} not found"}
    )

@app.exception_handler(RequestValidationError)
async def validation_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=422,
        content={"error": "VALIDATION_ERROR", "details": exc.errors()}
    )

@app.exception_handler(Exception)
async def general_handler(request: Request, exc: Exception):
    logger.error(f"Unhandled: {exc}", exc_info=True)
    return JSONResponse(status_code=500, content={"error": "INTERNAL_SERVER_ERROR"})

# Usage
@app.get("/users/{user_id}")
async def get_user(user_id: int, db: AsyncSession = Depends(get_db)):
    user = await crud_user.get(db, user_id)
    if not user:
        raise NotFoundException("User", user_id)
    return user
```

---

## 2.8 FASTAPI MIDDLEWARE

```python
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.gzip import GZipMiddleware
from starlette.middleware.base import BaseHTTPMiddleware
import time, uuid

app.add_middleware(CORSMiddleware,
    allow_origins=["https://myapp.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.add_middleware(GZipMiddleware, minimum_size=1000)

class TimingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start = time.perf_counter()
        request.state.request_id = str(uuid.uuid4())
        
        response = await call_next(request)
        
        duration = time.perf_counter() - start
        response.headers["X-Process-Time"] = f"{duration:.4f}"
        response.headers["X-Request-ID"] = request.state.request_id
        return response

app.add_middleware(TimingMiddleware)
```

---

## 2.9 FASTAPI LIFECYCLE EVENTS

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    # STARTUP — runs before app starts
    await database.connect()
    await redis.initialize()
    print("App started successfully")
    
    yield  # App is running
    
    # SHUTDOWN — runs when app stops
    await database.disconnect()
    await redis.close()
    print("App shut down cleanly")

app = FastAPI(lifespan=lifespan)
```

---

## 2.10 BACKGROUND TASKS

```python
from fastapi import BackgroundTasks

def send_notification(email: str, message: str):
    # Runs after response is sent — non-blocking
    send_email(email, message)

@app.post("/orders")
async def create_order(order: OrderCreate, background_tasks: BackgroundTasks):
    db_order = await crud.order.create(order)
    
    # Schedule background tasks (run AFTER response is sent)
    background_tasks.add_task(send_notification, order.email, "Order confirmed!")
    background_tasks.add_task(update_inventory, db_order.id)
    
    return db_order  # Response sent immediately, tasks run after

# Use Celery for:
# - Retries needed
# - Long-running tasks (>30 seconds)
# - Scheduled tasks
# - Monitoring/tracking needed

# Use BackgroundTasks for:
# - Simple, quick tasks
# - No retry needed
# - Fire and forget
```

---

## 2.11 FASTAPI TESTING

```python
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_create_user():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post("/users", json={
            "name": "Test User",
            "email": "test@example.com",
            "age": 25
        })
    assert response.status_code == 201
    assert response.json()["email"] == "test@example.com"

# Override dependencies for testing
async def override_get_db():
    yield test_db_session

app.dependency_overrides[get_db] = override_get_db

# conftest.py
@pytest.fixture
async def client():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac
```

---

## 2.12 FASTAPI RATE LIMITING

```python
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded

limiter = Limiter(key_func=get_remote_address, storage_uri="redis://localhost:6379")
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

@app.get("/api/data")
@limiter.limit("10/minute")
async def get_data(request: Request):
    return {"data": "..."}

@app.post("/auth/login")
@limiter.limit("5/minute")  # Stricter for auth endpoints
async def login(request: Request, form_data: OAuth2PasswordRequestForm = Depends()):
    ...
```

---

## 2.13 FASTAPI WEBSOCKETS

```python
from fastapi import WebSocket, WebSocketDisconnect

class ConnectionManager:
    def __init__(self):
        self.connections: dict[str, list[WebSocket]] = {}
    
    async def connect(self, ws: WebSocket, room: str):
        await ws.accept()
        self.connections.setdefault(room, []).append(ws)
    
    def disconnect(self, ws: WebSocket, room: str):
        self.connections[room].remove(ws)
    
    async def broadcast(self, message: str, room: str):
        for ws in self.connections.get(room, []):
            await ws.send_text(message)

manager = ConnectionManager()

@app.websocket("/ws/{room_id}")
async def websocket_endpoint(websocket: WebSocket, room_id: str):
    await manager.connect(websocket, room_id)
    try:
        while True:
            data = await websocket.receive_text()
            await manager.broadcast(f"Room {room_id}: {data}", room_id)
    except WebSocketDisconnect:
        manager.disconnect(websocket, room_id)
```

---

## 2.14 MONITORING WITH PROMETHEUS

```python
from prometheus_fastapi_instrumentator import Instrumentator

Instrumentator().instrument(app).expose(app, endpoint="/metrics")

# Metrics available at /metrics:
# http_requests_total
# http_request_duration_seconds
# http_requests_in_progress
# Run: docker run -p 9090:9090 prom/prometheus
# Configure Grafana to read from Prometheus
```

---

# ══════════════════════════════════════════
# PART 3: FLASK — COMPLETE GUIDE
# ══════════════════════════════════════════

---

## 3.1 WHAT IS FLASK?

**Simple Answer:**
Flask is a lightweight Python micro-framework. Unlike Django, it gives you just the essentials — routing and request handling. You add what you need.

**When to use Flask:**
- Small to medium projects
- Learning web development
- When you want full control over components
- Simple REST APIs
- Microservices

---

## 3.2 FLASK BASICS

```python
from flask import Flask, request, jsonify, g, abort
from functools import wraps

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your-secret-key'
app.config['DEBUG'] = False

# Basic route
@app.route('/')
def index():
    return 'Hello World!'

# Methods
@app.route('/users', methods=['GET', 'POST'])
def users():
    if request.method == 'GET':
        return jsonify([{"id": 1, "name": "Rahul"}])
    
    if request.method == 'POST':
        data = request.get_json()
        return jsonify({"id": 2, **data}), 201

# Path parameters
@app.route('/users/<int:user_id>', methods=['GET', 'PUT', 'DELETE'])
def user_detail(user_id):
    user = User.query.get_or_404(user_id)
    
    if request.method == 'GET':
        return jsonify(user.to_dict())
    elif request.method == 'PUT':
        data = request.get_json()
        user.update(**data)
        return jsonify(user.to_dict())
    elif request.method == 'DELETE':
        user.delete()
        return '', 204

if __name__ == '__main__':
    app.run(debug=True)
```

---

## 3.3 FLASK BLUEPRINTS

**Simple Explanation:**
Blueprints = modular parts of your Flask app. Like different departments in an office — HR, Finance, IT — each has its own files but works together.

```python
# users/routes.py
from flask import Blueprint, jsonify, request

users_bp = Blueprint('users', __name__, url_prefix='/api/users')

@users_bp.route('/')
def list_users():
    return jsonify({"users": []})

@users_bp.route('/<int:id>')
def get_user(id):
    return jsonify({"id": id})

# products/routes.py
products_bp = Blueprint('products', __name__, url_prefix='/api/products')

@products_bp.route('/')
def list_products():
    return jsonify({"products": []})

# app.py — register blueprints
from flask import Flask
from users.routes import users_bp
from products.routes import products_bp

def create_app():
    app = Flask(__name__)
    app.register_blueprint(users_bp)
    app.register_blueprint(products_bp)
    return app

app = create_app()
```

---

## 3.4 FLASK SQLAlchemy

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://user:pass@localhost/db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(120), unique=True, nullable=False)
    name = db.Column(db.String(80), nullable=False)
    is_active = db.Column(db.Boolean, default=True)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    orders = db.relationship('Order', backref='user', lazy=True)
    
    def to_dict(self):
        return {'id': self.id, 'email': self.email, 'name': self.name}

# CRUD
with app.app_context():
    db.create_all()
    
    # Create
    user = User(email='test@test.com', name='Test')
    db.session.add(user)
    db.session.commit()
    
    # Read
    users = User.query.filter_by(is_active=True).all()
    user = User.query.get(1)
    user = User.query.filter_by(email='test@test.com').first()
    
    # Update
    user.name = 'Updated'
    db.session.commit()
    
    # Delete
    db.session.delete(user)
    db.session.commit()
```

---

## 3.5 FLASK JWT AUTHENTICATION

```python
from flask import Flask, request, jsonify
from flask_jwt_extended import (
    JWTManager, create_access_token, create_refresh_token,
    jwt_required, get_jwt_identity, get_current_user
)
from datetime import timedelta

app = Flask(__name__)
app.config['JWT_SECRET_KEY'] = 'your-secret'
app.config['JWT_ACCESS_TOKEN_EXPIRES'] = timedelta(hours=1)
app.config['JWT_REFRESH_TOKEN_EXPIRES'] = timedelta(days=7)

jwt = JWTManager(app)

@app.route('/auth/login', methods=['POST'])
def login():
    data = request.get_json()
    user = User.query.filter_by(email=data['email']).first()
    
    if not user or not user.check_password(data['password']):
        return jsonify({"error": "Invalid credentials"}), 401
    
    return jsonify({
        "access_token": create_access_token(identity=user.id),
        "refresh_token": create_refresh_token(identity=user.id)
    })

@app.route('/auth/refresh', methods=['POST'])
@jwt_required(refresh=True)
def refresh():
    user_id = get_jwt_identity()
    return jsonify({"access_token": create_access_token(identity=user_id)})

@app.route('/profile')
@jwt_required()
def profile():
    user_id = get_jwt_identity()
    user = User.query.get(user_id)
    return jsonify(user.to_dict())
```

---

## 3.6 FLASK ERROR HANDLING

```python
from flask import Flask, jsonify

app = Flask(__name__)

class AppError(Exception):
    def __init__(self, message, status_code=400, error_code="APP_ERROR"):
        self.message = message
        self.status_code = status_code
        self.error_code = error_code

@app.errorhandler(404)
def not_found(e):
    return jsonify({"error": "NOT_FOUND", "message": "Resource not found"}), 404

@app.errorhandler(500)
def internal_error(e):
    return jsonify({"error": "INTERNAL_SERVER_ERROR", "message": str(e)}), 500

@app.errorhandler(AppError)
def handle_app_error(e):
    return jsonify({"error": e.error_code, "message": e.message}), e.status_code

# Usage
@app.route('/users/<int:id>')
def get_user(id):
    user = User.query.get(id)
    if not user:
        raise AppError(f"User {id} not found", 404, "USER_NOT_FOUND")
    return jsonify(user.to_dict())
```

---

## 3.7 FLASK MIDDLEWARE (Before/After Request)

```python
import time
import uuid
from flask import g, request

@app.before_request
def before_request():
    g.request_id = str(uuid.uuid4())
    g.start_time = time.perf_counter()
    
    # Auth check
    token = request.headers.get('Authorization', '').replace('Bearer ', '')
    if token:
        g.current_user = verify_token(token)

@app.after_request
def after_request(response):
    elapsed = time.perf_counter() - g.get('start_time', time.perf_counter())
    response.headers['X-Response-Time'] = f"{elapsed:.4f}"
    response.headers['X-Request-ID'] = g.get('request_id', '')
    return response

@app.teardown_appcontext
def teardown(exception):
    db.session.remove()  # Clean up DB session
```

---

## 3.8 FLASK MARSHMALLOW (Serialization)

```python
from flask_marshmallow import Marshmallow
from marshmallow import fields, validate, validates, ValidationError

ma = Marshmallow(app)

class UserSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = User
        exclude = ('password_hash',)
        load_instance = True
    
    email = fields.Email(required=True)
    name = fields.Str(required=True, validate=validate.Length(min=2, max=100))
    
    @validates('email')
    def validate_email(self, value):
        if User.query.filter_by(email=value).first():
            raise ValidationError("Email already registered")

user_schema = UserSchema()
users_schema = UserSchema(many=True)

@app.route('/users', methods=['POST'])
def create_user():
    try:
        user = user_schema.load(request.get_json())
        db.session.add(user)
        db.session.commit()
        return user_schema.dump(user), 201
    except ValidationError as e:
        return jsonify({"errors": e.messages}), 422
```

---

## 3.9 FLASK CACHING & CELERY

```python
from flask_caching import Cache
from celery import Celery

# Caching
app.config['CACHE_TYPE'] = 'RedisCache'
app.config['CACHE_REDIS_URL'] = 'redis://localhost:6379/0'
cache = Cache(app)

@app.route('/products')
@cache.cached(timeout=300, key_prefix='product_list')
def product_list():
    products = Product.query.all()
    return jsonify([p.to_dict() for p in products])

# Celery
def make_celery(app):
    celery = Celery(app.import_name)
    celery.conf.update(
        broker_url=app.config['CELERY_BROKER_URL'],
        result_backend=app.config['CELERY_RESULT_BACKEND']
    )
    
    class ContextTask(celery.Task):
        def __call__(self, *args, **kwargs):
            with app.app_context():
                return self.run(*args, **kwargs)
    
    celery.Task = ContextTask
    return celery

celery = make_celery(app)

@celery.task
def send_email(user_id):
    with app.app_context():
        user = User.query.get(user_id)
        # send actual email
        return f"Email sent to {user.email}"
```

---

## 3.10 FLASK TESTING

```python
import pytest
from app import create_app, db

@pytest.fixture
def app():
    app = create_app({'TESTING': True, 'SQLALCHEMY_DATABASE_URI': 'sqlite:///:memory:'})
    with app.app_context():
        db.create_all()
        yield app
        db.drop_all()

@pytest.fixture
def client(app):
    return app.test_client()

def test_list_users(client):
    response = client.get('/api/users/')
    assert response.status_code == 200
    assert isinstance(response.get_json(), list)

def test_create_user(client):
    response = client.post('/api/users/', json={
        "name": "Test",
        "email": "test@test.com",
        "password": "pass123"
    })
    assert response.status_code == 201
    assert response.get_json()['email'] == 'test@test.com'
```

---

# ══════════════════════════════════════════
# PART 4: COMPARISON TABLES
# ══════════════════════════════════════════

---

## Django vs FastAPI vs Flask

| Feature             | Django            | FastAPI           | Flask             |
|---------------------|-------------------|-------------------|-------------------|
| Type                | Full-stack        | API-focused       | Micro-framework   |
| Speed               | Medium            | Very Fast         | Fast              |
| ORM                 | Built-in          | SQLAlchemy        | SQLAlchemy/ext    |
| Admin Panel         | Built-in          | None              | Flask-Admin       |
| Auth                | Built-in          | Manual/libs       | Flask-Login       |
| Async               | Partial (3.1+)    | Native            | Partial (2.0+)    |
| Auto Docs           | No                | Yes (Swagger)     | Flask-RESTX/Spec  |
| Learning Curve      | Medium-High       | Medium            | Low               |
| Best for            | Full web apps     | Modern APIs       | Small APIs        |
| Validation          | Forms/Serializers | Pydantic          | Marshmallow       |
| WebSocket           | Channels          | Native            | Flask-SocketIO    |
| Migrations          | Built-in          | Alembic           | Flask-Migrate     |

---

## WSGI vs ASGI

| Feature         | WSGI                    | ASGI                         |
|-----------------|-------------------------|------------------------------|
| Type            | Synchronous             | Asynchronous                 |
| Concurrency     | One req at a time/worker| Many reqs concurrently       |
| Protocols       | HTTP only               | HTTP + WebSocket + HTTP/2    |
| Server          | Gunicorn                | Uvicorn, Hypercorn, Daphne   |
| Memory          | High (per process)      | Low (event loop)             |
| Use case        | CPU-bound, traditional  | I/O-bound, real-time         |

---

## Sync vs Async

| Aspect           | Synchronous              | Asynchronous               |
|------------------|--------------------------|----------------------------|
| Execution        | Blocking — waits for I/O | Non-blocking — continues   |
| Concurrency      | Via threads/processes    | Via coroutines             |
| Best for         | CPU-heavy tasks          | I/O-heavy (DB, APIs, files)|
| Memory           | Higher per request       | Lower per request          |
| Complexity       | Simpler                  | More complex to debug      |
| GIL impact       | Limited by GIL           | Bypasses GIL for I/O       |

---

## select_related vs prefetch_related

| Feature          | select_related           | prefetch_related            |
|------------------|--------------------------|-----------------------------|
| SQL queries      | 1 (JOIN)                 | 2 (separate, then merge)    |
| Best for         | ForeignKey, OneToOne     | ManyToMany, Reverse FK      |
| How it works     | SQL JOIN                 | Python-side join            |
| Memory           | Lower                    | Can be higher for M2M       |

---

## JWT vs Session Auth

| Feature          | JWT                      | Session                     |
|------------------|--------------------------|-----------------------------|
| Storage          | Client (header/cookie)   | Server (DB/Redis)           |
| Scalability      | Scales easily            | Needs shared session store  |
| Logout           | Complex (blacklist)      | Easy (delete session)       |
| Data             | Payload in token         | Just session ID             |
| Best for         | APIs, mobile, microservices | Traditional web apps      |
| Stateless        | Yes                      | No                          |

---

## APIView vs ViewSet

| Feature          | APIView                  | ViewSet                     |
|------------------|--------------------------|-----------------------------|
| Code amount      | More code                | Less code                   |
| URL setup        | Manual url patterns      | Auto via Router             |
| Flexibility      | Maximum                  | Convention-based            |
| Best for         | Custom, non-CRUD         | Standard CRUD               |
| Extra actions    | Separate class/function  | @action decorator           |

---

## Redis vs RabbitMQ

| Feature          | Redis                    | RabbitMQ                    |
|------------------|--------------------------|-----------------------------|
| Type             | In-memory data store     | Message broker              |
| Protocols        | Custom                   | AMQP                        |
| Use for Celery   | Simple/medium load       | High-load, complex routing  |
| Persistence      | Optional                 | Yes (message durability)    |
| Caching          | Yes (primary use)        | No                          |
| Setup            | Simple                   | Complex                     |
| Dead letter queue| Manual                   | Built-in                    |

---

## Gunicorn vs Uvicorn

| Feature          | Gunicorn                 | Uvicorn                     |
|------------------|--------------------------|-----------------------------|
| Protocol         | WSGI                     | ASGI                        |
| Type             | Synchronous workers      | Async event loop            |
| Best with        | Django, Flask            | FastAPI, Starlette           |
| Workers          | Fork-based               | Multiple event loops        |
| Production setup | Gunicorn only            | Gunicorn + Uvicorn workers  |

---

## Celery vs BackgroundTasks (FastAPI)

| Feature          | Celery                   | BackgroundTasks             |
|------------------|--------------------------|-----------------------------|
| Retry            | Built-in                 | Not supported               |
| Monitoring       | Flower, Redis            | None built-in               |
| Scheduled tasks  | Celery Beat              | Not supported               |
| Distributed      | Yes                      | No (same process)           |
| Setup complexity | High                     | Zero (built into FastAPI)   |
| Use when         | Long/complex tasks       | Quick fire-and-forget       |

---

# ══════════════════════════════════════════
# PART 5: SECURITY DEEP DIVE
# ══════════════════════════════════════════

---

## 5.1 JWT STEP-BY-STEP FLOW

```
1. Client → POST /auth/login {email, password}
2. Server → verify email+password from DB
3. Server → create JWT:
   - header: {"alg": "HS256", "typ": "JWT"}
   - payload: {"sub": user_id, "email": email, "exp": timestamp, "iat": timestamp}
   - signature: HMACSHA256(base64url(header)+"."+base64url(payload), SECRET_KEY)
4. Server → returns {access_token, refresh_token}
5. Client → stores tokens (access in memory, refresh in httpOnly cookie)
6. Client → every request: Authorization: Bearer <access_token>
7. Server → decode token (no DB lookup!) → extract user info
8. access_token expires → Client → POST /auth/refresh with refresh_token
9. Server → verify refresh_token → return new access_token
```

---

## 5.2 OAUTH2 FLOW

```
1. User clicks "Login with Google"
2. App redirects to Google (with client_id, redirect_uri, scope)
3. User logs in on Google, approves permissions
4. Google redirects back to app with authorization_code
5. App exchanges code for tokens (server-to-server, secret)
6. App receives access_token + refresh_token from Google
7. App uses access_token to call Google APIs for user info
8. App creates/updates user in own DB, creates own JWT
```

---

## 5.3 CSRF

**What:** Cross-Site Request Forgery — attacker tricks browser into making requests on user's behalf.

**How Django prevents it:**
- Generates unique CSRF token per session
- Embeds in forms and cookies
- Checks token on POST/PUT/DELETE

**When to disable:**
- API endpoints using JWT (token proves intent)

```python
# Django — safe for JWT APIs
from django.views.decorators.csrf import csrf_exempt
@csrf_exempt
def api_view(request): ...

# DRF handles automatically with SessionAuthentication vs TokenAuthentication
```

---

## 5.4 CORS

**What:** Cross-Origin Resource Sharing — controls which domains can call your API.

**Problem:** Browser blocks requests from `frontend.com` to `api.com` by default.

```python
# Django
CORS_ALLOWED_ORIGINS = ["https://myapp.com"]
CORS_ALLOW_CREDENTIALS = True

# FastAPI
app.add_middleware(CORSMiddleware,
    allow_origins=["https://myapp.com"],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
)

# Development only — allow all (NEVER in production!)
allow_origins=["*"]  # Dangerous in production!
```

---

## 5.5 PASSWORD HASHING

```python
# Django — uses PBKDF2 by default, Argon2 is better
from django.contrib.auth.hashers import make_password, check_password

hashed = make_password("rawpassword")  # Never store raw!
check_password("rawpassword", hashed)  # True

# Better: settings.py
PASSWORD_HASHERS = ['django.contrib.auth.hashers.Argon2PasswordHasher']

# FastAPI — passlib
from passlib.context import CryptContext
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

hashed = pwd_context.hash("rawpassword")
is_valid = pwd_context.verify("rawpassword", hashed)
```

---

## 5.6 RATE LIMITING

```python
# Django — django-ratelimit
from django_ratelimit.decorators import ratelimit

@ratelimit(key='ip', rate='10/m', block=True)
def my_view(request): ...

# FastAPI — slowapi
from slowapi import Limiter
limiter = Limiter(key_func=get_remote_address, storage_uri="redis://localhost:6379")

@app.post("/auth/login")
@limiter.limit("5/minute")
async def login(request: Request): ...

# Nginx level (most efficient)
# limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
# limit_req zone=api burst=20 nodelay;
```

---

## 5.7 SECRETS MANAGEMENT

```python
# NEVER hardcode secrets!

# Bad
SECRET_KEY = "my-secret-key-123"

# Good — environment variables
import os
SECRET_KEY = os.environ['SECRET_KEY']  # Raises error if missing

# Better — python-decouple
from decouple import config
SECRET_KEY = config('SECRET_KEY')

# Production — AWS Secrets Manager
import boto3
client = boto3.client('secretsmanager')
secret = client.get_secret_value(SecretId='myapp/prod/db')

# Use .env for development, NEVER commit to git
# Add .env to .gitignore!
```

---

## 5.8 SECURITY HEADERS

```python
# Django — django-csp or SecurityMiddleware
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_HSTS_SECONDS = 31536000
SECURE_SSL_REDIRECT = True
X_FRAME_OPTIONS = 'DENY'

# FastAPI — add via middleware
@app.middleware("http")
async def add_security_headers(request, call_next):
    response = await call_next(request)
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["X-XSS-Protection"] = "1; mode=block"
    response.headers["Strict-Transport-Security"] = "max-age=31536000"
    return response
```

---

# ══════════════════════════════════════════
# PART 6: DEPLOYMENT & DEVOPS
# ══════════════════════════════════════════

---

## 6.1 DOCKER

```dockerfile
# Dockerfile — Django / FastAPI
FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1

WORKDIR /app

RUN apt-get update && apt-get install -y libpq-dev gcc && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Security: non-root user
RUN adduser --disabled-password appuser && chown -R appuser /app
USER appuser

# Django
RUN python manage.py collectstatic --noinput

EXPOSE 8000

# Django command
CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "4"]

# FastAPI command
# CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

---

## 6.2 DOCKER COMPOSE

```yaml
# docker-compose.yml
version: '3.9'

services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: myapp_user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myapp_user"]
      interval: 10s
      retries: 5
  
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
  
  web:
    build: .
    depends_on:
      db:
        condition: service_healthy
    env_file: .env
    ports:
      - "8000:8000"
    volumes:
      - static:/app/static
  
  celery:
    build: .
    command: celery -A config worker --loglevel=info -c 4
    depends_on: [db, redis]
    env_file: .env
  
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      - static:/static
    depends_on: [web]

volumes:
  postgres_data:
  redis_data:
  static:
```

---

## 6.3 NGINX CONFIGURATION

```nginx
upstream app {
    server web:8000;
    keepalive 32;
}

server {
    listen 80;
    server_name myapp.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name myapp.com;
    
    ssl_certificate /etc/ssl/cert.pem;
    ssl_certificate_key /etc/ssl/key.pem;
    
    # Serve static files directly — much faster!
    location /static/ {
        alias /static/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    location / {
        proxy_pass http://app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Rate limiting
        limit_req zone=api burst=20 nodelay;
    }
    
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
}
```

---

## 6.4 GITHUB ACTIONS CI/CD

```yaml
# .github/workflows/ci.yml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test_db
        options: --health-cmd pg_isready --health-interval 10s
    
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - run: pip install -r requirements/testing.txt
      - run: flake8 . --max-line-length 100
      - run: pytest --cov=. --cov-report=xml -v
        env:
          DATABASE_URL: postgresql://postgres:test@localhost/test_db
          SECRET_KEY: test-secret-key
  
  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /app
            git pull origin main
            docker-compose up -d --build
            docker-compose exec web python manage.py migrate --noinput
```

---

## 6.5 PRODUCTION ARCHITECTURE

```
Internet
    ↓
Cloudflare (CDN + DDoS protection)
    ↓
Load Balancer (AWS ALB / Nginx)
    ↓
┌─────────────────────────────────────┐
│  Application Servers (Auto-scaling) │
│  Django/FastAPI + Gunicorn/Uvicorn  │
│  Server 1 | Server 2 | Server 3    │
└─────────────────────────────────────┘
    ↓
┌──────────┐  ┌──────────┐  ┌──────────────┐
│ Primary  │  │  Redis   │  │  Celery      │
│ DB (RDS) │  │  Cache   │  │  Workers     │
└──────────┘  └──────────┘  └──────────────┘
    ↓
Read Replicas (for SELECT queries)
    
┌──────────────────────────────────────┐
│  Monitoring Stack                    │
│  Prometheus → Grafana (metrics)      │
│  ELK Stack (logs)                    │
│  Sentry (error tracking)            │
└──────────────────────────────────────┘
```

---

# ══════════════════════════════════════════
# PART 7: DATABASE
# ══════════════════════════════════════════

---

## 7.1 INDEXING

```sql
-- B-Tree (default) — for =, >, <, BETWEEN, LIKE 'abc%'
CREATE INDEX idx_users_email ON users(email);

-- Composite index — order matters! (a, b) helps a alone, not b alone
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Partial index — only some rows (more efficient)
CREATE INDEX idx_active_users ON users(email) WHERE is_active = true;

-- GIN index — for arrays, JSONB, full-text search
CREATE INDEX idx_tags ON articles USING gin(tags);

-- Covering index — include extra columns
CREATE INDEX idx_email_cover ON users(email) INCLUDE (name, phone);

-- When NOT to index:
-- Low cardinality columns (gender: M/F — only 2 values, not worth it)
-- Write-heavy tables (index slows down INSERT/UPDATE/DELETE)
-- Small tables (full scan faster than index lookup)
```

**Django ORM indexing:**
```python
class Meta:
    indexes = [
        models.Index(fields=['email']),
        models.Index(fields=['user', 'status'], name='user_status_idx'),
        models.Index(fields=['created_at']),
    ]

# Field-level
email = models.EmailField(db_index=True)
```

---

## 7.2 TRANSACTIONS & ACID

```python
# Django transactions
from django.db import transaction

@transaction.atomic
def transfer_money(from_id, to_id, amount):
    from_acc = Account.objects.select_for_update().get(id=from_id)
    to_acc = Account.objects.select_for_update().get(id=to_id)
    
    if from_acc.balance < amount:
        raise ValueError("Insufficient balance")
    
    from_acc.balance -= amount
    to_acc.balance += amount
    from_acc.save()
    to_acc.save()
    # Commits only if no exception. Rolls back on ANY exception.

# Savepoints — nested transactions
def complex_op():
    with transaction.atomic():
        step_one()
        try:
            with transaction.atomic():  # Savepoint
                risky_step()
        except:
            pass  # Rolls back only inner transaction
        step_three()
```

---

## 7.3 N+1 QUERY PROBLEM

**Simple Explanation:**
Fetching 10 books = 1 query. Then accessing each book's author = 10 more queries.
Total = 11 queries. Should be 1.

```python
# ❌ N+1 Problem
books = Book.objects.all()        # 1 query
for book in books:
    print(book.author.name)       # 1 query × N books = N queries
# Total: 1 + N queries!

# ✅ Fixed with select_related
books = Book.objects.select_related('author').all()   # 1 JOIN query
for book in books:
    print(book.author.name)       # No extra query!

# Debug queries
from django.db import connection
print(len(connection.queries))    # Number of DB queries executed
```

---

## 7.4 CONNECTION POOLING

```python
# Django
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydb',
        'CONN_MAX_AGE': 60,  # Reuse connections for 60 seconds
    }
}

# Use PgBouncer in production for real connection pooling
# FastAPI — SQLAlchemy has built-in pooling
engine = create_async_engine(
    DATABASE_URL,
    pool_size=10,        # Normal connections
    max_overflow=20,     # Extra burst connections
    pool_pre_ping=True,  # Test connection before using
    pool_recycle=3600,   # Recycle connections after 1 hour
)
```

---

# ══════════════════════════════════════════
# PART 8: TOP INTERVIEW QUESTIONS
# ══════════════════════════════════════════

---

## DJANGO — TOP 100 QUESTIONS

### Basics

**Q1: What is Django? Why use it?**
A: Django is a high-level Python web framework with "batteries included" — built-in ORM, auth, admin, CSRF protection. Used by Instagram, Pinterest. Choose Django for rapid development of full-featured web apps.

**Q2: What is the Django MTV pattern?**
A: Model (data/DB layer), Template (HTML/presentation), View (business logic). Unlike MVC, Django's View is the controller.

**Q3: What is manage.py?**
A: CLI tool that wraps Django's management commands. Used for runserver, migrate, createsuperuser, shell, collectstatic.

**Q4: What is the difference between project and app in Django?**
A: Project = entire website configuration. App = specific feature/module (users, products). One project can have many apps.

**Q5: Explain Django's request-response cycle.**
A: Request → WSGI Server → Middleware (top-down) → URL Router → View → Model → Template/JSON → Middleware (bottom-up) → Response.

### ORM

**Q6: What is ORM?**
A: Object Relational Mapper. Converts Python code to SQL. `User.objects.filter(is_active=True)` → `SELECT * FROM users WHERE is_active=TRUE`.

**Q7: What is a QuerySet?**
A: Lazy collection of DB objects. Doesn't execute SQL until evaluated (iterated, sliced, or calling list(), count(), etc.).

**Q8: Difference between get() and filter()?**
A: `get()` returns one object, raises DoesNotExist or MultipleObjectsReturned. `filter()` returns QuerySet (can be empty), never raises.

**Q9: What is select_related?**
A: Uses SQL JOIN to fetch FK/OneToOne related objects in one query. Solves N+1 for single relationships.

**Q10: What is prefetch_related?**
A: Makes 2 queries — one for main objects, one for related objects — Python joins them. For ManyToMany and reverse FK.

**Q11: What is the N+1 query problem?**
A: Fetching N objects and then querying each one's relation = N+1 queries. Solve with select_related/prefetch_related.

**Q12: What are F() expressions?**
A: Reference DB field values without loading into Python.
`Product.objects.update(price=F('price') * 1.1)` — 1 DB query, no Python calculation.

**Q13: What are Q() objects?**
A: Complex OR/AND queries: `User.objects.filter(Q(email__contains='@gmail') | Q(email__contains='@yahoo'))`.

**Q14: Difference between annotate() and aggregate()?**
A: `annotate()` adds calculated field to each object in QuerySet. `aggregate()` returns single summary dict for entire QuerySet.

**Q15: What is select_for_update()?**
A: Locks selected rows until transaction ends. Prevents race conditions. `Account.objects.select_for_update().get(id=1)`.

### Views

**Q16: FBV vs CBV — when to use which?**
A: FBV for simple, custom logic. CBV for reusable CRUD — use mixins and inheritance to avoid repetition.

**Q17: What are generic views in Django?**
A: Pre-built CBVs for common patterns — ListView, DetailView, CreateView, UpdateView, DeleteView. Reduce boilerplate.

**Q18: What is a mixin in Django?**
A: A class that provides specific functionality to be combined with other classes. LoginRequiredMixin, PermissionRequiredMixin, etc.

### DRF

**Q19: What is DRF?**
A: Django REST Framework — toolkit for building Web APIs on top of Django. Adds serializers, authentication, permissions, pagination.

**Q20: Difference between APIView, GenericAPIView, ViewSet?**
A: APIView — full control, write everything. GenericAPIView — adds queryset/serializer_class. ViewSet — router-compatible, less code. ModelViewSet — CRUD for free.

**Q21: What is a serializer in DRF?**
A: Converts complex data (models) to/from Python dicts/JSON. Also handles validation.

**Q22: Difference between Serializer and ModelSerializer?**
A: ModelSerializer auto-generates fields from model. Serializer you define all fields manually. ModelSerializer saves ~70% code for CRUD.

**Q23: How does DRF authentication work?**
A: Multiple backends: Session, Token, JWT, OAuth2. Set `DEFAULT_AUTHENTICATION_CLASSES` in settings. Can mix multiple.

**Q24: What is the difference between authentication and authorization?**
A: Authentication = WHO are you (identity). Authorization = WHAT can you do (permissions).

**Q25: How do you create custom permissions in DRF?**
A: Extend `BasePermission`, implement `has_permission(request, view)` and/or `has_object_permission(request, view, obj)`.

### Models

**Q26: null=True vs blank=True?**
A: `null=True` → DB stores NULL. `blank=True` → form/serializer accepts empty string. For CharField: use blank=True. For FK: use null=True.

**Q27: What is abstract model?**
A: Model with `abstract = True` — no DB table, just shared fields for child models. Best for mixins like TimeStampedModel.

**Q28: What is proxy model?**
A: Same table as parent, different Python class. Used to add custom managers/methods or change Meta without schema changes.

**Q29: What is multi-table inheritance?**
A: Each model in hierarchy gets its own table, linked via OneToOne. Allows polymorphism but adds JOIN overhead.

**Q30: Why must custom User model be created at project start?**
A: Django's migrations system references AUTH_USER_MODEL. Changing it later breaks all existing migrations and foreign keys.

### Security

**Q31: What is CSRF and how does Django prevent it?**
A: Cross-Site Request Forgery. Django generates CSRF token, includes in forms/cookies, validates on unsafe methods. Disable for JWT APIs.

**Q32: What is XSS and how does Django prevent it?**
A: Cross-Site Scripting — inject malicious JS. Django templates auto-escape HTML. Never use `|safe` on user input.

**Q33: How does Django prevent SQL injection?**
A: ORM always parameterizes queries. Never use string formatting in raw SQL — use `%s` parameters instead.

**Q34: What is CORS?**
A: Cross-Origin Resource Sharing. Controls which domains can call your API. Use `django-cors-headers` to configure.

**Q35: How does Django store passwords?**
A: Hashed with PBKDF2 by default. Can configure Argon2 (recommended). Never stores raw passwords.

### Middleware

**Q36: What is middleware in Django?**
A: Hooks that process every request and response. Runs top-down for requests, bottom-up for responses. Use for auth, logging, rate limiting.

**Q37: How do you write custom middleware?**
A: Create class with `__init__(get_response)` and `__call__(request)`. Add to MIDDLEWARE list in settings.

**Q38: Difference between middleware and signals?**
A: Middleware = request/response level, runs on every HTTP request. Signals = model event level, triggered by specific model events.

### Caching

**Q39: What caching levels exist in Django?**
A: Per-site, per-view (@cache_page), template fragment ({% cache %}), low-level API (cache.get/set).

**Q40: How do you use Redis as Django cache?**
A: `pip install django-redis`. Set CACHES backend to `django_redis.cache.RedisCache` with Redis URL.

### Celery

**Q41: What is Celery?**
A: Distributed task queue for running async/background tasks. Uses broker (Redis/RabbitMQ) to communicate with workers.

**Q42: What is a Celery broker?**
A: Message middleware that holds tasks until workers pick them up. Redis for simple setups, RabbitMQ for advanced routing.

**Q43: How do you retry failed Celery tasks?**
A: Use `bind=True` and `self.retry(exc=exc, countdown=60)`. Set `max_retries`. Use `autoretry_for` for automatic retry.

**Q44: What is Celery Beat?**
A: Celery's scheduler. Runs periodic tasks (like cron). Use `celery -A app beat`. Store schedules in DB with django-celery-beat.

**Q45: Difference between .delay() and .apply_async()?**
A: `.delay()` is shortcut for `.apply_async()`. `apply_async` gives more control: countdown, eta, queue, priority.

### Testing

**Q46: How do you test Django APIs?**
A: Use `APITestCase` with `APIClient`. `force_authenticate()` for auth. Use factories (factory_boy) for test data.

**Q47: What is pytest-django?**
A: Pytest plugin for Django. Use `@pytest.mark.django_db` for DB access. Better fixtures and test discovery than unittest.

### Performance

**Q48: How do you find slow queries in Django?**
A: `django-debug-toolbar` shows all queries in dev. `django-silk` for profiling. Enable SQL logging in settings. Use EXPLAIN ANALYZE in PostgreSQL.

**Q49: What is CONN_MAX_AGE?**
A: Time (seconds) Django keeps DB connections open. Avoids connection overhead. Set to 60-120 for production.

**Q50: What are the main Django performance optimizations?**
A: select_related/prefetch_related, indexes, caching (Redis), only()/defer(), bulk operations, connection pooling (CONN_MAX_AGE), async tasks (Celery).

### Deployment

**Q51: What is Gunicorn?**
A: WSGI HTTP server for Django. Manages multiple worker processes. `gunicorn config.wsgi:application -w 4`.

**Q52: How many Gunicorn workers should you use?**
A: Rule: (2 × CPU cores) + 1. For 4 cores = 9 workers. Too many = memory pressure.

**Q53: Why use Nginx with Django?**
A: Serves static files (much faster). Reverse proxy to Gunicorn. SSL termination. Load balancing. Rate limiting.

**Q54: What is the difference between DEBUG=True and DEBUG=False?**
A: DEBUG=True shows detailed error pages, serves static files. DEBUG=False hides errors (shows 500), requires Nginx for static, required in production.

**Q55: How do you manage Django settings for different environments?**
A: Split into base.py, development.py, production.py. Set `DJANGO_SETTINGS_MODULE` env variable. Use `python-decouple` for secrets.

### Advanced

**Q56: What are Django Channels?**
A: Extension for handling WebSockets, long-polling. Uses ASGI instead of WSGI. Allows real-time features.

**Q57: How does Django's async support work?**
A: Django 3.1+ supports async views. Mark with `async def`. Use `await` for DB (via `sync_to_async`). Full async ORM in Django 4.1+.

**Q58: What is Django's content types framework?**
A: Allows FK to any model dynamically. Used for generic relations (e.g., comments on any object type).

**Q59: What is Django admin and how to customize it?**
A: Auto-generated admin UI. Customize with ModelAdmin class — add list_display, search_fields, list_filter, custom actions.

**Q60: What are Django management commands?**
A: Custom CLI commands via `python manage.py mycommand`. Extend `BaseCommand`. Good for data migrations, scheduled scripts.

**Q61: How do you handle file uploads in production Django?**
A: Store in AWS S3 using `django-storages`. Never store in same server as app code. Set `DEFAULT_FILE_STORAGE`.

**Q62: What is Django's contenttypes framework?**
A: Creates FK to any model. Used for generic relations (like- button on any object type, activity feeds, audit logs).

**Q63: Explain Django's session framework.**
A: Stores user data between requests. Default: DB backend. Can use file, cache (Redis), cookie-based. `request.session['key'] = value`.

**Q64: What are class-based views advantages?**
A: Code reuse via inheritance. Mixins for cross-cutting concerns. Less repetition for CRUD. Organized by HTTP method.

**Q65: What is difference between OneToOneField and ForeignKey?**
A: OneToOne = exactly one related object (unique FK). ForeignKey = many objects can point to one (many-to-one).

**Q66: What are through tables in M2M?**
A: Extra model for M2M with additional fields.
`tags = ManyToManyField(Tag, through='ArticleTag')`.

**Q67: What is prefetch_related Prefetch object?**
A: Fine-grained prefetch control — custom queryset, to_attr for storing in custom attribute.

**Q68: How to create a read replica setup in Django?**
A: Use multiple DATABASES and `DATABASE_ROUTERS`. Router decides which DB to use for read vs write.

**Q69: What is Django's check framework?**
A: Validation system that checks for common errors. Run with `python manage.py check`. Runs automatically before runserver/migrate.

**Q70: How does Django migrations handle dependencies?**
A: Each migration lists `dependencies` on previous migrations. Django builds a directed graph and runs in correct order.

**Q71-100** (quick answers):

| # | Question | Short Answer |
|---|----------|--------------|
|71| What is lazy vs eager loading? | Lazy=fetch when needed (QuerySet). Eager=fetch now (select_related) |
|72| What is database sharding? | Split large DB across multiple servers by key |
|73| How to handle DB connection errors? | `CONN_MAX_AGE=0` or catch `OperationalError`, retry |
|74| What is Django's FormView? | CBV for form handling with GET/POST support |
|75| What is ModelForm? | Auto-generates form from model fields |
|76| How to do bulk upsert in Django? | `bulk_create(update_conflicts=True)` in Django 4.1+ |
|77| What is `update_or_create`? | Updates if exists, creates if not. Returns (obj, created) |
|78| What is `get_or_create`? | Gets if exists, creates if not. Returns (obj, created) |
|79| What is `values()` vs `values_list()`? | values()=list of dicts. values_list()=list of tuples |
|80| Explain `only()` vs `defer()` | only() fetches specified. defer() fetches all except specified |
|81| What are database views in Django? | Unmanaged models pointing to SQL views (`managed = False`) |
|82| What is `iterator()` in QuerySet? | Streams results — low memory, can't be cached/reused |
|83| How to test Celery tasks? | `task.apply()` runs synchronously in tests |
|84| What is `django.test.override_settings`? | Temporarily override settings in tests |
|85| How to add custom admin actions? | Define method, add to `actions` list in ModelAdmin |
|86| What is `InlineModelAdmin`? | Shows related objects inline in parent admin page |
|87| What is `django-extensions`? | Useful management commands: shell_plus, runscript, graph_models |
|88| What is `cached_property`? | Property cached on first access. Per-instance cache |
|89| How to schedule tasks without Celery? | `django-crontab`, management commands + system cron |
|90| What is `GenericForeignKey`? | FK to any model using ContentType framework |
|91| What is `UUIDField`? | Primary key using UUID instead of integer |
|92| How to encrypt fields in Django? | `django-encrypted-model-fields`, `pgcrypto` extension |
|93| What is `DatabaseError`? | Base exception for all DB errors |
|94| How to log all SQL queries? | Set `django.db.backends` logger to DEBUG level |
|95| What is `connection.queries`? | List of all executed queries (DEBUG=True only) |
|96| How to use raw SQL safely? | `Model.objects.raw(sql, [params])` or `connection.execute(sql, [params])` |
|97| What is `db_table` in Meta? | Custom DB table name instead of app_modelname |
|98| What is `db_column` in field? | Custom column name instead of field name |
|99| What is `app_label` in Meta? | Specify which app a model belongs to |
|100| How to handle soft deletes? | Add `is_deleted` BooleanField + custom manager filtering it out |

---

## FASTAPI — TOP 100 QUESTIONS

**Q1: What is FastAPI?**
A: Modern Python web framework for building APIs. Built on Starlette (ASGI), uses Pydantic for validation, type hints for auto-docs.

**Q2: Why is FastAPI fast?**
A: ASGI-native (async), Pydantic v2 validation (Rust core), no ORM overhead, Starlette underneath, minimal abstraction.

**Q3: What is Pydantic?**
A: Data validation library. Define schemas with Python type hints. Validates, serializes, and deserializes data automatically.

**Q4: What is Dependency Injection in FastAPI?**
A: `Depends()` injects pre-computed values into endpoints — DB sessions, current user, pagination params. Clean, reusable, testable.

**Q5: What is the difference between path and query parameters?**
A: Path: `/users/{id}` — part of URL. Query: `/users?page=1&limit=20` — after `?` in URL.

**Q6: What is response_model?**
A: Pydantic model that shapes API response — filters out extra fields, validates output. Different from return type hint.

**Q7: How do you handle errors in FastAPI?**
A: `raise HTTPException(status_code=404, detail="Not found")`. Global handlers via `@app.exception_handler(ExceptionType)`.

**Q8: What is lifespan in FastAPI?**
A: Async context manager for startup/shutdown events. Connect DB on startup, close on shutdown.

**Q9: BackgroundTasks vs Celery?**
A: BackgroundTasks — simple, no retry, same process. Celery — distributed, retries, monitoring, scheduled tasks.

**Q10: How does async work in FastAPI?**
A: `async def` endpoints don't block event loop during I/O. Run in asyncio event loop. All concurrent in one thread.

**Q11-50** (key answers):

| # | Question | Short Answer |
|---|----------|--------------|
|11| What is ASGI? | Async Server Gateway Interface — supports HTTP + WebSocket + HTTP/2 |
|12| What is Uvicorn? | ASGI server for FastAPI. Production: Gunicorn + UvicornWorker |
|13| What is `status_code` in response? | HTTP status. 200 OK, 201 Created, 204 No Content, 400 Bad, 401 Unauth |
|14| How to validate request body? | Pydantic BaseModel with Field validators |
|15| What is `Field(...)` in Pydantic? | ... = required. Adds constraints: min_length, ge, le, regex |
|16| How to implement OAuth2? | `OAuth2PasswordBearer`, verify token in Depends() |
|17| How to add middleware? | `app.add_middleware(ClassName, **params)` |
|18| How to upload files? | `UploadFile = File(...)` parameter. Read with `await file.read()` |
|19| How to stream responses? | `StreamingResponse(generator(), media_type="text/plain")` |
|20| What is `Annotated` in FastAPI? | Combine type hint + validation: `Annotated[int, Field(ge=0)]` |
|21| How to version APIs? | URL prefix: `/api/v1/`, `/api/v2/` with separate routers |
|22| How to add custom response headers? | `Response` object with `response.headers["X-Key"] = "value"` |
|23| What is `include_router`? | Attach APIRouter to main app with prefix/tags |
|24| How to handle CORS? | `CORSMiddleware` with `allow_origins` |
|25| How to test FastAPI? | `AsyncClient` from `httpx` with `app` and `base_url` |
|26| What is `model_dump()` in Pydantic v2? | Converts model to dict. Previously `.dict()` |
|27| How to disable docs in production? | `FastAPI(docs_url=None, redoc_url=None, openapi_url=None)` |
|28| What is `Depends` caching? | Same dependency in same request = called once, result shared |
|29| How to override dependency in tests? | `app.dependency_overrides[original] = mock_function` |
|30| What is `yield` in dependency? | Allows cleanup code after endpoint finishes (for DB sessions) |
|31| What is `HTTPException`? | FastAPI's built-in exception → auto JSON response |
|32| How to add request ID? | Middleware sets `request.state.request_id` → attach to response header |
|33| How to paginate in FastAPI? | Class-based `PaginationParams` dependency with offset/limit |
|34| What is `response_model_exclude_unset`? | Only include fields that were set, not defaults |
|35| How to add tags to endpoints? | `@app.get("/", tags=["Users"])` — groups in Swagger UI |
|36| What is `APIRouter`? | Mini-app with its own routes/dependencies. Attach to main app |
|37| How to add custom OpenAPI schema? | `app.openapi()` override or add `example` to Pydantic fields |
|38| What is `alias` in Pydantic? | Map JSON field name to Python attr: `Field(alias="user_name")` |
|39| How to handle None in response? | `response_model_exclude_none=True` removes None fields |
|40| What is `ConfigDict` in Pydantic v2? | Model-level config. `from_attributes=True` for ORM mode |
|41| Difference between `@validator` and `@field_validator`? | `@validator` = Pydantic v1. `@field_validator` = Pydantic v2 |
|42| How to add auth to Swagger UI? | Add `SecurityScheme` in OpenAPI config with `http bearer` |
|43| What is `computed_field`? | Derived Pydantic field computed from other fields |
|44| How to log requests? | Middleware that logs method, path, status, duration |
|45| What is `model_validator`? | Cross-field validation in Pydantic v2 |
|46| How to handle large file uploads? | Stream upload with `await file.read(chunk_size)` |
|47| What is `Form()` in FastAPI? | Parse form data (`Content-Type: application/x-www-form-urlencoded`) |
|48| How to return HTML from FastAPI? | `HTMLResponse(content="<html>...</html>")` |
|49| What is `Starlette`? | ASGI toolkit FastAPI is built on. Handles routing, middleware, WebSockets |
|50| How to add health check? | `@app.get("/health") → return {"status": "ok"}` |

**Q51-100** (quick answers):

| # | Question | Short Answer |
|---|----------|--------------|
|51| Sync vs Async endpoint? | `async def` for I/O-bound. `def` for CPU-bound (runs in threadpool) |
|52| What is `asyncio.gather()`? | Run multiple coroutines concurrently |
|53| How to use Redis in FastAPI? | `aioredis` for async. Cache DB results, rate limit, sessions |
|54| What is `alembic`? | DB migration tool for SQLAlchemy (equivalent to Django migrations) |
|55| How to run Alembic? | `alembic revision --autogenerate -m "msg"` then `alembic upgrade head` |
|56| What is `declarative_base()`? | Base class for SQLAlchemy models |
|57| How to handle DB errors? | Try/except in endpoint, rollback in `finally`, log error |
|58| What is `expire_on_commit=False`? | Keep objects loaded after commit (needed for async) |
|59| How to do soft delete in FastAPI? | Set `is_deleted=True` instead of delete. Filter in get_queryset |
|60| How to add rate limiting? | `slowapi` library with Redis backend |
|61| What is Prometheus? | Metrics collection. Expose `/metrics`, scrape with Prometheus, visualize in Grafana |
|62| How to structured log in FastAPI? | `python-json-logger`, log in JSON format with correlation IDs |
|63| What is `connection_pool`? | SQLAlchemy pool manages DB connections — reuses them efficiently |
|64| What are WebSocket states? | CONNECTING → OPEN → CLOSING → CLOSED |
|65| How to broadcast WebSocket? | ConnectionManager maintains active_connections dict, broadcast to all |
|66| What is `select_in_loading`? | SQLAlchemy's equivalent of prefetch_related for async |
|67| What is `lazy="selectin"` in relationship? | Auto-load related objects using SELECT IN query |
|68| How to test WebSocket? | `async with AsyncClient() as client: ws = client.websocket_connect(url)` |
|69| What is `APIRouter prefix`? | Common URL prefix for all routes in router: `APIRouter(prefix="/api/v1/users")` |
|70| How to add custom serializer? | Custom `json_encoders` in Pydantic model Config |
|71| What is `model_config`? | Pydantic v2 model configuration via `ConfigDict` |
|72| How to implement RBAC? | Permission levels in JWT claims, check in Depends() |
|73| What is `Annotated` type? | `Annotated[str, Field(max_length=100)]` — type + validation info |
|74| How to use environment variables? | `BaseSettings` reads from `.env` automatically |
|75| What is `Security()` vs `Depends()`? | Security() same as Depends() but shows in OpenAPI schema |
|76| How to add database indexes? | `Column(index=True)` or `__table_args__ = (Index(...),)` in SQLAlchemy |
|77| What is `selectinload` in SQLAlchemy? | Eager loading strategy — SELECT IN for relationships |
|78| How to handle timeouts? | `asyncio.wait_for(coroutine(), timeout=30.0)` |
|79| What is `httpx`? | Async HTTP client. Used for testing FastAPI and calling external APIs |
|80| How to add API key auth? | `APIKeyHeader` from `fastapi.security` |
|81| What is `Jinja2Templates`? | Server-side HTML templates in FastAPI |
|82| How to return 204 No Content? | `Response(status_code=204)` or `status_code=HTTP_204_NO_CONTENT` |
|83| What is `exclude_unset` in Pydantic? | Only validate/return fields explicitly set by client (for PATCH) |
|84| How to do partial updates (PATCH)? | Use `model.model_dump(exclude_unset=True)` to get only changed fields |
|85| What is `aiofiles`? | Async file I/O library for FastAPI file operations |
|86| How to add request validation globally? | Custom middleware or Pydantic base models |
|87| What is `mount()`? | Mount static files or other ASGI apps into FastAPI |
|88| How to serve static files? | `app.mount("/static", StaticFiles(directory="static"))` |
|89| What is `TestClient` vs `AsyncClient`? | TestClient = sync testing. AsyncClient = async testing (preferred) |
|90| How to debug FastAPI? | Use `print()`, `logging`, VS Code debugger, or `--reload` for hot reload |
|91| What is API gateway pattern? | Single entry point routing to multiple microservices |
|92| How to implement microservices with FastAPI? | Separate FastAPI apps per service, communicate via HTTP/message queue |
|93| What is `gRPC`? | Binary protocol for service communication, faster than REST |
|94| How to handle circular imports? | Use `TYPE_CHECKING`, lazy imports, or restructure code |
|95| What is `__all__` in Python? | Controls what `from module import *` exports |
|96| How to profile FastAPI? | `pyinstrument`, `cProfile`, APM tools (Datadog, New Relic) |
|97| What is `Hypercorn`? | Alternative ASGI server supporting HTTP/2 |
|98| How to handle graceful shutdown? | lifespan context manager, catch SIGTERM signal |
|99| What is `py.test --asyncio-mode=auto`? | Auto-detect async tests without `@pytest.mark.asyncio` |
|100| FastAPI vs Django REST Framework? | FastAPI: faster, async, auto-docs, type hints. DRF: mature, admin, batteries-included |

---

## FLASK — TOP 100 QUESTIONS

**Q1: What is Flask?**
A: Lightweight Python micro-framework. Minimalist — gives routing + request/response. Add extensions for everything else.

**Q2: Flask vs Django vs FastAPI?**
A: Flask = simple, flexible, you choose components. Django = full-stack, everything included. FastAPI = fast, async, auto-docs.

**Q3: What is WSGI in Flask?**
A: Web Server Gateway Interface. Flask is WSGI app. Use `gunicorn` as WSGI server in production.

**Q4: What is application factory pattern?**
A: `create_app()` function that creates/configures Flask app. Better for testing (create fresh app per test).

**Q5: What are Blueprints?**
A: Modular components. Define routes, error handlers separately. Register with `app.register_blueprint(bp)`.

**Q6: What is `g` in Flask?**
A: Request-context global. Store per-request data (current user, request ID). Cleared after each request.

**Q7: What is `request` in Flask?**
A: Global proxy to current HTTP request. `request.json`, `request.args`, `request.form`, `request.headers`.

**Q8: What is Flask-SQLAlchemy?**
A: SQLAlchemy integration for Flask. `db = SQLAlchemy(app)`. Models extend `db.Model`.

**Q9: What is app context in Flask?**
A: Application-level context (not request-specific). Access with `with app.app_context():`. Needed for DB, email, etc. outside request.

**Q10: What is `before_request` and `after_request`?**
A: Hook functions. `before_request` runs before every request. `after_request` runs after (must return response).

**Q11-50:**

| # | Question | Short Answer |
|---|----------|--------------|
|11| What is `url_for()`? | Generates URL for a view function by name |
|12| What is `redirect()`? | Returns redirect response: `return redirect(url_for('index'))` |
|13| What is `abort()`? | Raises HTTP error: `abort(404)`, `abort(403)` |
|14| What is Jinja2? | Template engine. `{{ variable }}`, `{% for %}`, `{% if %}` |
|15| What is Flask-Migrate? | Alembic integration for Flask. `flask db migrate`, `flask db upgrade` |
|16| What is Flask-Login? | User session management. `login_user()`, `logout_user()`, `@login_required` |
|17| What is Flask-JWT-Extended? | JWT auth for Flask. `create_access_token()`, `@jwt_required()` |
|18| What is Flask-CORS? | CORS support: `CORS(app, origins=["https://myapp.com"])` |
|19| What is Flask-Caching? | Cache integration. `@cache.cached(timeout=300)` |
|20| What is Marshmallow? | Serialization/deserialization/validation library for Flask APIs |
|21| What is `teardown_appcontext`? | Cleanup after request context — close DB sessions |
|22| What is `errorhandler`? | `@app.errorhandler(404)` — handle specific HTTP errors |
|23| What is `jsonify()`? | Returns JSON response with correct Content-Type header |
|24| How to handle CORS in Flask? | `flask-cors`: `CORS(app)` or per-blueprint |
|25| What is Flask-WTF? | WTForms integration — CSRF protection for forms |
|26| What is `test_client()`? | Flask test client for writing tests without running server |
|27| How to set cookie in Flask? | `response.set_cookie('key', 'value', httponly=True)` |
|28| How to get cookie in Flask? | `request.cookies.get('key')` |
|29| What is Flask session? | Server-side session via signed cookie. `session['user'] = user_id` |
|30| What is `SECRET_KEY` in Flask? | Used to sign session cookies and CSRF tokens. Required for sessions |
|31| How to do pagination in Flask? | `Model.query.paginate(page=1, per_page=20)` |
|32| What is Flask-Admin? | Auto-generates admin UI for SQLAlchemy models |
|33| What is `config.from_object()`? | Load config from Python object/class |
|34| What is `config.from_envvar()`? | Load config from environment variable pointing to config file |
|35| How to handle file uploads in Flask? | `request.files['file']`. `file.save()` to save to disk. Validate extension |
|36| What is `send_file()`? | Return a file as response. `send_file('path/to/file.pdf')` |
|37| What is Flask-Mail? | Email sending extension. `mail.send_message(...)` |
|38| What is Flask-RESTful? | Extension for REST APIs. `Resource` classes with `get()`, `post()` |
|39| What is Flask-RESTX? | Enhanced Flask-RESTful with Swagger UI |
|40| What is `@property` in Flask model? | Computed attribute on model class |
|41| How to implement search in Flask? | SQLAlchemy `.filter(Model.name.ilike(f'%{query}%'))` |
|42| What is `current_app` proxy? | Access Flask app from within request context |
|43| What is `LocalProxy`? | Thread-local proxy — makes `request`, `g`, `current_app` thread-safe |
|44| How to connect to multiple DBs? | Multiple `SQLAlchemy` instances or SQLAlchemy binds |
|45| What is `db.session.flush()`? | Write to DB without commit (gets generated IDs) |
|46| What is `db.session.expunge()`? | Remove object from session (won't be tracked) |
|47| What is `db.session.merge()`? | Copy state from detached object into session |
|48| How to do raw SQL in Flask-SQLAlchemy? | `db.session.execute(text("SELECT..."), {"param": value})` |
|49| What is `backref` in relationship? | Auto-creates reverse relationship on related model |
|50| What is `lazy='joined'` in relationship? | Load related objects with JOIN (like select_related) |

**Q51-100:**

| # | Question | Short Answer |
|---|----------|--------------|
|51| What is `lazy='select'` in relationship? | Load related objects when accessed (default, N+1 risk) |
|52| What is `lazy='subquery'` in relationship? | Use subquery to load related objects |
|53| How to implement soft delete? | `is_deleted` flag + custom `query_class` filter |
|54| What is Flask Signal? | `blinker` library signals. `user_logged_in.connect(handler)` |
|55| What is `MethodView`? | CBV for Flask. Methods match HTTP verbs: `def get()`, `def post()` |
|56| How to add rate limiting? | `flask-limiter` extension |
|57| What is `before_first_request`? | Deprecated. Use `with app.app_context()` in create_app() |
|58| How to structure large Flask apps? | Application factory + Blueprints + Services + Repositories |
|59| What is Alembic? | DB migration tool. Flask-Migrate wraps it |
|60| How to run Flask in production? | `gunicorn "app:create_app()" -w 4 -b 0.0.0.0:8000` |
|61| What is `TESTING = True` in Flask? | Disables error catching — shows full traceback in tests |
|62| What is `WTF_CSRF_ENABLED = False`? | Disable CSRF for API tests |
|63| How to implement OAuth in Flask? | `flask-oauthlib` or `authlib` library |
|64| What is `flask shell`? | Interactive shell with app context pre-loaded |
|65| What is `flask routes`? | CLI command to show all registered routes |
|66| How to debug Flask? | `app.run(debug=True)` enables debugger + auto-reload |
|67| What is Werkzeug? | WSGI utility library Flask is built on. Handles routing, requests |
|68| What is `click` in Flask? | CLI framework. `flask` commands use click |
|69| How to add custom CLI commands? | `@app.cli.command('mycommand')` or `@click.command()` |
|70| What is Flask-APScheduler? | Add APScheduler for background scheduled tasks |
|71| What is `db.event.listen()`? | SQLAlchemy ORM events (before_insert, after_update, etc.) |
|72| How to handle 413 Request Entity Too Large? | Set `MAX_CONTENT_LENGTH = 16 * 1024 * 1024` (16MB) |
|73| What is `send_from_directory()`? | Safely serve files from directory |
|74| How to implement API versioning? | Blueprint per version: `v1 = Blueprint('v1', prefix='/api/v1')` |
|75| What is Flask-Principal? | Role and identity management |
|76| How to add health check? | `@app.route('/health') → return jsonify({"status": "ok"})` |
|77| What is context local? | Thread-local (or async-local) storage for request/app context |
|78| What is `push_appcontext()`? | Manually push app context in scripts/tests |
|79| How to handle long running tasks? | Celery + Redis/RabbitMQ |
|80| What is Flask-SocketIO? | WebSocket support for Flask via Socket.IO |
|81| How to implement caching? | `flask-caching` with Redis backend |
|82| What is `environ` in Flask? | WSGI environ dict — raw request data |
|83| How to return paginated response? | `paginate()` returns Pagination object with items, total, pages |
|84| What is `strict_slashes`? | If False, `/users` and `/users/` both work |
|85| What is `host_matching`? | Route to different apps based on hostname |
|86| How to test with DB? | Use in-memory SQLite + `db.create_all()` in fixture |
|87| What is `ImmutableDict`? | Flask uses it for request.args — read-only |
|88| What is `dataclasses` support in Flask? | `jsonify()` supports dataclasses in newer versions |
|89| How to do input sanitization? | `bleach` library for HTML, whitelist validation |
|90| What is `PREFERRED_URL_SCHEME`? | Force https in `url_for()` output |
|91| What is `SERVER_NAME`? | Used for subdomain matching and URL generation |
|92| How to add logging? | `app.logger.info()`, configure `logging.basicConfig()` |
|93| What is `after_this_request`? | Modify response for just the current request |
|94| What is request hooking order? | before_request → view → after_request → teardown |
|95| What is Flask async support? | Flask 2.0+ supports `async def` views with asyncio |
|96| How to stream responses? | `stream_with_context(generator())` |
|97| What is `PROPAGATE_EXCEPTIONS`? | If True, exceptions propagate to test client |
|98| What is `EXPLAIN` in SQLAlchemy? | `db.session.execute(text("EXPLAIN ANALYZE ..."))` |
|99| How to add Swagger to Flask? | `flask-restx` or `flasgger` or `apispec` |
|100| Flask production checklist? | DEBUG=False, SECRET_KEY from env, Gunicorn+Nginx, HTTPS, Redis cache, Celery tasks, logging |

---

## HR QUESTIONS

**Q: Tell me about yourself.**
A: "I'm a Python backend developer with X years of experience, specializing in Django and FastAPI. I've built [mention key projects]. I'm passionate about building scalable APIs and writing clean, testable code."

**Q: What's your biggest technical achievement?**
A: Use STAR format. Mention specific metrics: "Reduced API response time by 80% by implementing Redis caching and fixing N+1 queries."

**Q: Why do you want to join our company?**
A: Research the company. Mention their tech stack, scale challenges, mission. Show you've read about their engineering blog.

**Q: What's your weakness?**
A: Mention something real but improvable: "I sometimes over-engineer — I now use simpler solutions first and refactor when needed."

**Q: How do you handle disagreements with teammates?**
A: "I focus on data and outcomes rather than opinions. I document the trade-offs and escalate to tech lead if needed."

---

## SCENARIO-BASED QUESTIONS

**Q: Your API suddenly takes 30 seconds. What do you do?**
```
1. Check: Is it ALL endpoints or ONE specific? 
2. Check logs: Any errors? Timeouts?
3. Check resources: CPU, Memory, DB connections, Redis
4. Check DB: Long-running queries? Lock contention?
5. Add timing: Which line in code is slow?
6. Fix: N+1? Missing index? Cache miss? Memory leak?
7. Verify: Metrics improve after fix
8. Post-mortem: Document and prevent recurrence
```

**Q: How would you migrate 5 million records without downtime?**
```
1. Add new column (nullable) in separate migration
2. Deploy code that writes to BOTH old + new column
3. Run background Celery task to backfill new column in batches
4. Monitor backfill progress
5. Deploy code that only reads new column
6. Drop old column in next release
```

**Q: Design a URL shortener API.**
```
POST /shorten → body: {url} → returns {short_code, short_url}
GET /{short_code} → 301 redirect to original URL

Design:
- Store mapping: {short_code: original_url} in Redis (fast) + DB (persistent)
- Generate short_code: Base62 encode incrementing ID
- Cache: Most accessed codes in Redis (cache-aside)
- Rate limit: 10 shorten requests per minute per IP
- Analytics: Track clicks with Celery task
```

---

# ══════════════════════════════════════════
# PART 9: CODING EXAMPLES
# ══════════════════════════════════════════

---

## CRUD API — FastAPI

```python
from fastapi import FastAPI, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession
from pydantic import BaseModel
from typing import List, Optional

app = FastAPI()

class ProductCreate(BaseModel):
    name: str
    price: float
    description: Optional[str] = None

class ProductUpdate(BaseModel):
    name: Optional[str] = None
    price: Optional[float] = None
    description: Optional[str] = None

class ProductResponse(BaseModel):
    id: int
    name: str
    price: float
    model_config = {"from_attributes": True}

@app.post("/products", response_model=ProductResponse, status_code=201)
async def create_product(product: ProductCreate, db: AsyncSession = Depends(get_db)):
    db_product = Product(**product.model_dump())
    db.add(db_product)
    await db.commit()
    await db.refresh(db_product)
    return db_product

@app.get("/products", response_model=List[ProductResponse])
async def list_products(skip: int = 0, limit: int = 20, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Product).offset(skip).limit(limit))
    return result.scalars().all()

@app.get("/products/{id}", response_model=ProductResponse)
async def get_product(id: int, db: AsyncSession = Depends(get_db)):
    product = await db.get(Product, id)
    if not product:
        raise HTTPException(404, "Product not found")
    return product

@app.patch("/products/{id}", response_model=ProductResponse)
async def update_product(id: int, update: ProductUpdate, db: AsyncSession = Depends(get_db)):
    product = await db.get(Product, id)
    if not product:
        raise HTTPException(404, "Product not found")
    for key, value in update.model_dump(exclude_unset=True).items():
        setattr(product, key, value)
    await db.commit()
    await db.refresh(product)
    return product

@app.delete("/products/{id}", status_code=204)
async def delete_product(id: int, db: AsyncSession = Depends(get_db)):
    product = await db.get(Product, id)
    if not product:
        raise HTTPException(404, "Product not found")
    await db.delete(product)
    await db.commit()
```

---

## JWT AUTH — FastAPI (complete flow)

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import jwt, JWTError
from passlib.context import CryptContext
from datetime import datetime, timedelta

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"

pwd_context = CryptContext(schemes=["bcrypt"])
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/token")

def create_token(user_id: int, expires: timedelta) -> str:
    payload = {"sub": str(user_id), "exp": datetime.utcnow() + expires}
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

async def get_current_user(token: str = Depends(oauth2_scheme), db = Depends(get_db)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id = int(payload["sub"])
    except (JWTError, KeyError):
        raise HTTPException(401, "Invalid token", headers={"WWW-Authenticate": "Bearer"})
    user = await db.get(User, user_id)
    if not user or not user.is_active:
        raise HTTPException(401, "User not found or inactive")
    return user

@app.post("/auth/token")
async def login(form: OAuth2PasswordRequestForm = Depends(), db = Depends(get_db)):
    result = await db.execute(select(User).where(User.email == form.username))
    user = result.scalar_one_or_none()
    if not user or not pwd_context.verify(form.password, user.hashed_password):
        raise HTTPException(401, "Incorrect email or password")
    return {
        "access_token": create_token(user.id, timedelta(minutes=60)),
        "refresh_token": create_token(user.id, timedelta(days=7)),
        "token_type": "bearer"
    }

@app.get("/me")
async def get_me(user: User = Depends(get_current_user)):
    return {"id": user.id, "email": user.email}
```

---

## REDIS CACHING PATTERN

```python
import json
import redis.asyncio as aioredis

redis_client = aioredis.from_url("redis://localhost:6379")

async def get_cached(key: str):
    data = await redis_client.get(key)
    return json.loads(data) if data else None

async def set_cached(key: str, data, ttl: int = 300):
    await redis_client.setex(key, ttl, json.dumps(data, default=str))

async def invalidate(key: str):
    await redis_client.delete(key)

# Usage in FastAPI
@app.get("/products/{id}")
async def get_product(id: int, db = Depends(get_db)):
    cache_key = f"product:{id}"
    
    cached = await get_cached(cache_key)
    if cached:
        return cached
    
    product = await crud.product.get(db, id)
    if not product:
        raise HTTPException(404)
    
    data = ProductResponse.from_orm(product).dict()
    await set_cached(cache_key, data, 300)
    return data
```

---

## CELERY TASK WITH RETRY

```python
from celery import shared_task
from celery.utils.log import get_task_logger

logger = get_task_logger(__name__)

@shared_task(
    bind=True,
    max_retries=5,
    default_retry_delay=60,
    autoretry_for=(ConnectionError, TimeoutError),
    retry_backoff=True,
)
def send_order_notification(self, order_id: int, user_email: str):
    try:
        order = Order.objects.get(id=order_id)
        result = email_service.send(
            to=user_email,
            subject=f"Order #{order_id} Confirmed",
            template="order_confirmation",
            context={"order": order}
        )
        logger.info(f"Email sent: order_id={order_id}")
        return {"status": "sent", "email": user_email}
    
    except Order.DoesNotExist:
        logger.error(f"Order {order_id} not found")
        return {"status": "failed", "reason": "not_found"}
    
    except Exception as exc:
        logger.warning(f"Retry #{self.request.retries}: {exc}")
        raise self.retry(exc=exc, countdown=2 ** self.request.retries)

# Calling
send_order_notification.delay(order_id=123, user_email="user@example.com")
```

---

## PAGINATION EXAMPLE

```python
# Django DRF
class ProductListView(generics.ListAPIView):
    serializer_class = ProductSerializer
    pagination_class = PageNumberPagination

# FastAPI
from fastapi import Query
from typing import Generic, TypeVar, List
from pydantic import BaseModel

T = TypeVar("T")

class Page(BaseModel, Generic[T]):
    items: List[T]
    total: int
    page: int
    page_size: int
    pages: int

@app.get("/products", response_model=Page[ProductResponse])
async def list_products(
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
    db = Depends(get_db)
):
    offset = (page - 1) * page_size
    
    total_result = await db.execute(select(func.count(Product.id)))
    total = total_result.scalar()
    
    result = await db.execute(select(Product).offset(offset).limit(page_size))
    items = result.scalars().all()
    
    return {
        "items": items,
        "total": total,
        "page": page,
        "page_size": page_size,
        "pages": (total + page_size - 1) // page_size
    }
```

---

## LOGGING SETUP

```python
# logging_config.py
import logging
import sys
from pythonjsonlogger import jsonlogger

def setup_logging(log_level: str = "INFO"):
    logger = logging.getLogger()
    logger.setLevel(getattr(logging, log_level))
    
    handler = logging.StreamHandler(sys.stdout)
    formatter = jsonlogger.JsonFormatter(
        fmt="%(asctime)s %(levelname)s %(name)s %(message)s",
        datefmt="%Y-%m-%dT%H:%M:%S"
    )
    handler.setFormatter(formatter)
    logger.addHandler(handler)

# Usage
logger = logging.getLogger(__name__)

logger.info("User logged in", extra={"user_id": 123, "ip": "192.168.1.1"})
logger.error("Payment failed", extra={"order_id": 456, "error_code": "CARD_DECLINED"})
```

---

# ══════════════════════════════════════════
# PART 10: SYSTEM DESIGN BASICS
# ══════════════════════════════════════════

---

## 10.1 SCALABILITY PATTERNS

```
Vertical Scaling: Bigger server (more RAM, CPU) — limited, expensive
Horizontal Scaling: More servers — preferred, load balanced

Load Balancing:
- Round robin: Request 1→Server1, Request 2→Server2, ...
- Least connections: Route to server with fewest active connections
- IP hash: Same user always goes to same server (session stickiness)

Database Scaling:
- Read replicas: Write to primary, read from replicas
- Sharding: Split data across multiple DBs (user_id % 4 = which shard)
- Connection pooling: PgBouncer, SQLAlchemy pool
```

---

## 10.2 MICROSERVICES COMMUNICATION

```
Synchronous (HTTP):
Service A → POST /api/order → Service B (waits for response)
+ Simple, easy to debug
- Tight coupling, failure propagation

Asynchronous (Message Queue):
Service A → Publishes "order.created" event → RabbitMQ → Service B consumes
+ Decoupled, resilient, scalable
- More complex, eventual consistency

API Gateway Pattern:
Client → API Gateway → [User Service, Order Service, Payment Service]
Gateway handles: Auth, Rate limiting, Routing, SSL termination, Logging
```

---

## 10.3 QUICK SYSTEM DESIGN CHECKLIST

For ANY system design question, cover:
1. **Functional requirements** — What must the system do?
2. **Non-functional requirements** — Scale, availability, latency
3. **API design** — Endpoints, request/response
4. **Database design** — Schema, indexes, choices (SQL vs NoSQL)
5. **Caching** — What, where, TTL, invalidation
6. **Scaling** — Load balancer, horizontal scaling, sharding
7. **Async processing** — Celery/queue for heavy tasks
8. **Monitoring** — Metrics, logging, alerting

---

## FINAL CHEAT SHEET — QUICK REVISION

### Django Must-Remember
```
N+1 → select_related (FK) / prefetch_related (M2M)
Signals → post_save, pre_save (use sparingly)
ACID → @transaction.atomic + select_for_update()
Security → CSRF, XSS (auto-escaped), parameterized queries
Caching → cache.get/set, @cache_page, Redis backend
Celery → .delay(), bind=True, max_retries, retry backoff
JWT → simplejwt, access token (60min), refresh (7days)
Performance → only(), defer(), bulk_create(), CONN_MAX_AGE
```

### FastAPI Must-Remember
```
Pydantic → validation, BaseSettings, Field(...), validators
Depends() → DB session, auth, pagination — composable
async def → for I/O. def → for CPU (runs in threadpool)
lifespan → startup/shutdown events
BackgroundTasks → quick tasks. Celery → complex/retryable
exception_handler → global error handling
response_model → output shaping/filtering
Middleware → BaseHTTPMiddleware, add_middleware()
```

### Flask Must-Remember
```
create_app() → application factory pattern
Blueprints → modular routes organization
g → per-request storage
before_request / after_request → middleware hooks
Flask-SQLAlchemy → db.Model, db.session, db.Column
Flask-JWT-Extended → @jwt_required(), create_access_token()
@app.errorhandler(404) → custom error pages
gunicorn "app:create_app()" → production server
```

### Security Must-Remember
```
JWT = header.payload.signature → stateless auth
CSRF = token in form/cookie → protect state-changing requests
XSS = escape HTML output → Django does this auto
SQL injection = parameterized queries → ORM does this auto
CORS = allow specific origins only → never * in production
Rate limiting = per-IP, per-user → nginx or slowapi
Password = bcrypt/argon2 → never store plaintext
Secrets = env vars or secrets manager → never in code
```

---

# 🏆 INTERVIEW SUCCESS TIPS

**1. Think out loud:** Say what you're thinking. Interviewers evaluate your process, not just the answer.

**2. Always mention trade-offs:** "I'd use Redis caching here, but that adds infrastructure complexity..."

**3. Bring up production concerns:** Testing, logging, monitoring, scalability, security — unprompted.

**4. Use metrics in answers:** "This reduced response time from 8s to 180ms" > "This made it faster"

**5. Know your WHY:** Not just HOW to use something, but WHY — when to use it vs alternatives.

**6. Admit what you don't know:** "I haven't used that specific tool, but I've used similar X and would approach it like..."

**7. Ask clarifying questions for design problems:** Shows senior thinking.

**8. Practice these 5 answers out loud:**
- "Tell me about a complex API you built"
- "How did you handle performance issues in production?"
- "How do you approach testing?"
- "Explain a technical decision and its trade-offs"
- "How would you design [URL shortener / notification system / auth service]?"

---

*Complete Backend Interview Guide — Django • FastAPI • Flask*
*Covers Beginner → Expert | 400+ Questions | Production Ready*
