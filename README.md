**Django Basics** using a real-world example: a **User Management App** (where users can register, view a list, and see individual user profiles). We’ll cover:

---

## ✅ Django Basics Explained with User Management App:

### Topics Covered:

1. Django Models  
2. Django Views  
3. Django URLs  
4. Django Templates  
5. Django ORM  
6. Django Admin  

---

## 🛠 Step-by-Step Project: **User Management App**

---

### ✅ Step 1: Project & App Setup

```bash
django-admin startproject userproject
cd userproject
python manage.py startapp accounts
```

Add `'accounts'` to `INSTALLED_APPS` in `userproject/settings.py`:

```python
INSTALLED_APPS = [
    ...
    'accounts',
]
```

---

### ✅ Step 2: Django Models – Define User Profile

Go to `accounts/models.py` and create a simple `UserProfile` model:

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

---

### ✅ Step 3: Django ORM – Migrations

Create and apply migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

✅ **Django ORM** allows you to interact with the database using Python (not raw SQL). For example:

```python
UserProfile.objects.create(name="Anthor", email="anthor@example.com", age=29)
users = UserProfile.objects.all()
```

---

### ✅ Step 4: Django Admin – Register Model

Go to `accounts/admin.py`:

```python
from django.contrib import admin
from .models import UserProfile

admin.site.register(UserProfile)
```

Create superuser and access the admin:

```bash
python manage.py createsuperuser
```

Visit: `http://127.0.0.1:8000/admin` → Login → You’ll see UserProfile section.

---

### ✅ Step 5: Django Views – Handle Logic

Go to `accounts/views.py`:

```python
from django.shortcuts import render
from .models import UserProfile

def home(request):
    return render(request, 'accounts/home.html')

def user_list(request):
    users = UserProfile.objects.all()
    return render(request, 'accounts/user_list.html', {'users': users})

def user_detail(request, user_id):
    user = UserProfile.objects.get(id=user_id)
    return render(request, 'accounts/user_detail.html', {'user': user})
```

---

### ✅ Step 6: Django URLs – Routing

Create a new file: `accounts/urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
    path('users/', views.user_list, name='user_list'),
    path('users/<int:user_id>/', views.user_detail, name='user_detail'),
]
```

Now link this to your main `userproject/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('accounts.urls')),
]
```

---

### ✅ Step 7: Django Templates – HTML Pages

Inside the `accounts` folder, create a `templates/accounts/` folder structure like this:

```bash
mkdir -p accounts/templates/accounts
```

Then create the following HTML files:

---

#### 🧾 `home.html`

```html
<h1>Welcome to User Management App</h1>
<a href="{% url 'user_list' %}">View All Users</a>
```

---

#### 🧾 `user_list.html`

```html
<h2>All Users</h2>
<ul>
  {% for user in users %}
    <li>
      <a href="{% url 'user_detail' user.id %}">{{ user.name }}</a>
    </li>
  {% endfor %}
</ul>
```

---

#### 🧾 `user_detail.html`

```html
<h2>User Detail</h2>
<p><strong>Name:</strong> {{ user.name }}</p>
<p><strong>Email:</strong> {{ user.email }}</p>
<p><strong>Age:</strong> {{ user.age }}</p>
<p><strong>Joined:</strong> {{ user.joined_on }}</p>
<a href="{% url 'user_list' %}">Back to list</a>
```

---

## ✅ Summary of Concepts

| Topic        | How We Used It                         |
|--------------|----------------------------------------|
| **Models**   | Defined `UserProfile` in `models.py`   |
| **Views**    | Created logic in `views.py`            |
| **URLs**     | Mapped paths to views                  |
| **Templates**| Rendered HTML for each view            |
| **ORM**      | Used `.all()`, `.get()` for DB queries |
| **Admin**    | Registered model to manage via admin   |

---

Would you like me to turn this into a GitHub repo for you to try?

If everything is clear, we’ll move to the **DRF version of this same app** using API + JWT next. Let me know!
