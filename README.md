Awesome Anthor! Let’s go step-by-step and **see the output + understand what is happening at every stage** of our **User Management App using Django Basics**.

---

## ✅ Step-by-Step with Output & Explanation:

---

### ✅ Step 1: Project & App Creation

```bash
django-admin startproject userproject
cd userproject
python manage.py startapp accounts
```

🧠 **What’s happening?**
- You created a Django project named `userproject`.
- Inside that, you created an app called `accounts` which will handle our user features.

✅ **Output:**
You'll see folders like:

```
userproject/
├── accounts/
├── userproject/
```

---

### ✅ Step 2: Register `accounts` App in settings

Open `userproject/settings.py`, go to `INSTALLED_APPS` and add:

```python
'accounts',
```

🧠 **Why?**
So Django knows to include this app in the project.

✅ **Output:**
No visible output yet, but this is mandatory for models to work.

---

### ✅ Step 3: Create the Model

Open `accounts/models.py`:

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

🧠 **Explanation:**
- You’ve defined a model `UserProfile` with name, email, age, and join date.
- `__str__` makes it readable in admin/list views.

---

### ✅ Step 4: Make & Apply Migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

🧠 **What’s happening?**
- `makemigrations` creates migration files from your model.
- `migrate` applies them to the database.

✅ **Output:**

```bash
Migrations for 'accounts':
  accounts/migrations/0001_initial.py
    - Create model UserProfile
```

---

### ✅ Step 5: Register the Model in Admin

Open `accounts/admin.py`:

```python
from django.contrib import admin
from .models import UserProfile

admin.site.register(UserProfile)
```

Create a superuser:

```bash
python manage.py createsuperuser
```

✅ **Now visit:**  
`http://127.0.0.1:8000/admin`  
Login → You’ll see:

```
User Profiles
```

Click it → You can **add**, **edit**, and **delete** users via UI.

---

### ✅ Step 6: Create Views

Open `accounts/views.py`:

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

🧠 **Explanation:**
- `home`: static welcome page
- `user_list`: gets all users from DB using ORM
- `user_detail`: fetches a user by ID

✅ **Django ORM Usage:**
```python
UserProfile.objects.all()         # Get all users
UserProfile.objects.get(id=1)     # Get user with ID = 1
```

---

### ✅ Step 7: Define URLs

Create `accounts/urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
    path('users/', views.user_list, name='user_list'),
    path('users/<int:user_id>/', views.user_detail, name='user_detail'),
]
```

Now link it in `userproject/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('accounts.urls')),
]
```

🧠 **What’s happening?**
You’re telling Django which view should handle which URL.

✅ **Try in browser:**

- `http://127.0.0.1:8000/` → Home Page
- `http://127.0.0.1:8000/users/` → List of all users
- `http://127.0.0.1:8000/users/1/` → Details of user with ID 1

---

### ✅ Step 8: Create Templates

Create the folder structure:

```bash
mkdir -p accounts/templates/accounts
```

---

#### 📄 home.html

```html
<h1>Welcome to User Management App</h1>
<a href="{% url 'user_list' %}">View All Users</a>
```

✅ Output:
```
Welcome to User Management App
[View All Users]
```

---

#### 📄 user_list.html

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

✅ Output:
```
All Users
- Anthor
- Jony
- Sakib
```
(Each name is a link to user details)

---

#### 📄 user_detail.html

```html
<h2>User Detail</h2>
<p><strong>Name:</strong> {{ user.name }}</p>
<p><strong>Email:</strong> {{ user.email }}</p>
<p><strong>Age:</strong> {{ user.age }}</p>
<p><strong>Joined:</strong> {{ user.joined_on }}</p>
<a href="{% url 'user_list' %}">Back to list</a>
```

✅ Output (for user ID 1):
```
User Detail
Name: Anthor
Email: anthor@example.com
Age: 29
Joined: 2025-03-21 16:23
```

---

## ✅ Summary: What You Learned with Real Output

| Concept        | Output/Result                                                                 |
|----------------|-------------------------------------------------------------------------------|
| **Model**      | Created `UserProfile` table in DB                                             |
| **View**       | Rendered pages using database info                                            |
| **URL**        | Mapped URLs like `/users/1/` to specific views                                |
| **Template**   | Showed user info in styled HTML                                               |
| **ORM**        | Used `.all()`, `.get()` to access data                                        |
| **Admin**      | Admin panel shows and manages `UserProfile` entries easily                   |

---

Let me know once this is 100% clear. Then I’ll take you to the **same app using Django REST Framework (DRF) + JWT authentication**, step-by-step.

Would you like me to zip this project or upload to GitHub too?
