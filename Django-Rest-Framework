Amazing Anthor! You're thinking like a real developer now 🔥. I’ll now walk you through the **entire DRF + JWT User Management App** step-by-step again — **explaining each line of code**, the **purpose behind it**, and what the **output looks like**.

---

## 🧠 Our Goal:
A fully working **User Management API** using:
- Django REST Framework
- JWT Authentication
- CRUD operations via API
- Filtering, Pagination, Ordering
- Secure endpoints using permissions

---

# ✅ STEP-BY-STEP EXPLANATION + OUTPUT

---

## ✅ Step 1: Install Required Packages

```bash
pip install djangorestframework djangorestframework-simplejwt django-filter
```

### 🔍 Why?
- `djangorestframework`: for building APIs
- `simplejwt`: for token-based JWT auth
- `django-filter`: to filter data via API (like `?age=30`)

---

## ✅ Step 2: Configure Installed Apps and DRF Settings

In `settings.py`:

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework_simplejwt',
    'django_filters',
    'accounts',
]
```

Now add the DRF settings:

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    ),
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 5,
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
        'rest_framework.filters.OrderingFilter',
        'rest_framework.filters.SearchFilter',
    ]
}
```

### 🔍 Why?
- All APIs are **JWT-protected**
- Pagination = 5 users per page
- APIs will support filtering, ordering, and searching

---

## ✅ Step 3: Create the Model

In `accounts/models.py`:

```python
from django.db import models

class UserProfile(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    age = models.IntegerField()
    joined_on = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.name
```

### 🔍 What this does:
- `name`, `email`, `age`: user info
- `joined_on`: set automatically when a record is created
- `__str__`: makes admin/CLI output readable

```bash
python manage.py makemigrations
python manage.py migrate
```

✅ **Output:** Model is now stored in the database as a table.

---

## ✅ Step 4: Create the Serializer

In `accounts/serializers.py`:

```python
from rest_framework import serializers
from .models import UserProfile

class UserProfileSerializer(serializers.ModelSerializer):
    class Meta:
        model = UserProfile
        fields = '__all__'
```

### 🔍 What this does:
Serializers convert:
- **Model → JSON** (for API response)
- **JSON → Model** (for creating/updating via POST/PUT)

---

## ✅ Step 5: Create ViewSet

In `accounts/views.py`:

```python
from rest_framework import viewsets, permissions
from .models import UserProfile
from .serializers import UserProfileSerializer

class UserProfileViewSet(viewsets.ModelViewSet):
    queryset = UserProfile.objects.all()
    serializer_class = UserProfileSerializer
    permission_classes = [permissions.IsAuthenticated]
    filterset_fields = ['age']
    ordering_fields = ['name', 'joined_on']
    search_fields = ['name', 'email']
```

### 🔍 What this does:
- `ModelViewSet` gives full CRUD support
- `permission_classes` ensures JWT token is required
- `filterset_fields`, `ordering_fields`, `search_fields` enable advanced query features

---

## ✅ Step 6: Create the API Router

In `accounts/api_urls.py`:

```python
from rest_framework.routers import DefaultRouter
from .views import UserProfileViewSet

router = DefaultRouter()
router.register(r'users', UserProfileViewSet)

urlpatterns = router.urls
```

### 🔍 Why this is powerful:
Router automatically creates:
- `GET /api/users/`
- `POST /api/users/`
- `GET /api/users/1/`
- `PUT /api/users/1/`
- `DELETE /api/users/1/`

No manual URL writing needed for CRUD.

---

## ✅ Step 7: JWT Auth & Main URL Setup

In `userproject/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include
from accounts import api_urls
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include(api_urls)),
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]
```

---

## ✅ Step 8: Test JWT Authentication (Using Postman)

### 1️⃣ **Login & Get Token**

```bash
POST http://localhost:8000/api/token/
Content-Type: application/json

{
  "username": "admin",
  "password": "yourpassword"
}
```

✅ **Response:**

```json
{
  "access": "your-access-token",
  "refresh": "your-refresh-token"
}
```

---

### 2️⃣ **Access Protected API**

```bash
GET http://localhost:8000/api/users/
Authorization: Bearer your-access-token
```

✅ **Output:**
```json
{
  "count": 12,
  "next": "http://localhost:8000/api/users/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "name": "Anthor",
      "email": "anthor@example.com",
      "age": 29,
      "joined_on": "2025-03-22T18:00:00Z"
    },
    ...
  ]
}
```

---

## ✅ Extra Features

### 🔍 Search

```
GET /api/users/?search=anthor
```

### 🔍 Filter

```
GET /api/users/?age=29
```

### 🔍 Order

```
GET /api/users/?ordering=-joined_on
```

---

## ✅ Custom Exception Handling (Optional)

In `accounts/views.py`:

```python
from rest_framework.views import exception_handler

def custom_exception_handler(exc, context):
    response = exception_handler(exc, context)
    if response is not None:
        response.data['message'] = str(exc)
        response.data['status_code'] = response.status_code
    return response
```

In `settings.py`:

```python
REST_FRAMEWORK['EXCEPTION_HANDLER'] = 'accounts.views.custom_exception_handler'
```

✅ **Now if any error occurs**, you’ll get cleaner, informative JSON messages like:

```json
{
  "message": "Authentication credentials were not provided.",
  "status_code": 401
}
```

---

## 🧭 Full API Workflow Summary

| Action         | URL                            | Method | Auth? | Description                    |
|----------------|--------------------------------|--------|-------|--------------------------------|
| Get token      | `/api/token/`                  | POST   | ❌    | Login to get JWT               |
| Refresh token  | `/api/token/refresh/`          | POST   | ❌    | Refresh access token           |
| List users     | `/api/users/`                  | GET    | ✅    | List all users (paginated)     |
| Create user    | `/api/users/`                  | POST   | ✅    | Add a new user                 |
| Retrieve user  | `/api/users/1/`                | GET    | ✅    | Get a single user              |
| Update user    | `/api/users/1/`                | PUT    | ✅    | Full update                    |
| Delete user    | `/api/users/1/`                | DELETE | ✅    | Delete user                    |
| Search users   | `/api/users/?search=anthor`    | GET    | ✅    | Search by name/email           |
| Filter users   | `/api/users/?age=29`           | GET    | ✅    | Filter by age                  |
| Order users    | `/api/users/?ordering=name`    | GET    | ✅    | Order by name or joined date   |

---

## ✅ Next Step Options

1. Upload full project to GitHub ✅  
2. Add **User Registration API** (custom user model) ✅  
3. Add **email verification / password reset** ✅  
4. Build **frontend React app** for this backend ✅

Let me know what you want next, or if you want me to zip the full project now 💻📁
