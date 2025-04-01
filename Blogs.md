Sure! Below is the full code for a **Django REST Framework** application that includes:

- **User Registration**
- **User Login**
- **Blog Creation**
- **Blog Viewing**
- **Blog Editing**

---

### **Step-by-Step Implementation:**

1. **Set up your Django project and app:**

```bash
django-admin startproject blog_project
cd blog_project
python manage.py startapp blog
```

2. **Install Django REST Framework:**

```bash
pip install djangorestframework
```

3. **Add Installed Apps in `settings.py`:**

In your `settings.py`, add `'rest_framework'` and your app `'blog'` to `INSTALLED_APPS`.

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'blog',
]
```

4. **Set Up the Custom User Model:**

Define a custom user model in `blog/models.py`.

```python
# blog/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    email = models.EmailField(unique=True)
    is_verified = models.BooleanField(default=False)
    otp = models.CharField(max_length=6, null=True, blank=True)

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']

    def __str__(self):
        return self.email
```

Make sure to set the custom user model in `settings.py`.

```python
# blog_project/settings.py
AUTH_USER_MODEL = 'blog.CustomUser'
```

5. **Create the Blog Model:**

Define the `Blog` model to handle blog creation and edits.

```python
# blog/models.py
from django.db import models
from django.conf import settings

class Blog(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title
```

6. **Create the Serializers:**

Define serializers for the **CustomUser** and **Blog** models in `blog/serializers.py`.

```python
# blog/serializers.py
from rest_framework import serializers
from .models import Blog
from django.contrib.auth import get_user_model

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = get_user_model()
        fields = ['id', 'username', 'email', 'first_name', 'last_name']

class BlogSerializer(serializers.ModelSerializer):
    author = UserSerializer(read_only=True)

    class Meta:
        model = Blog
        fields = ['id', 'title', 'content', 'author', 'created_at', 'updated_at']
```

7. **Create the Views with APIView:**

Use Django REST Framework's `APIView` for handling API requests.

```python
# blog/views.py
from rest_framework import status
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.views import APIView
from .models import Blog
from .serializers import BlogSerializer
from django.shortcuts import get_object_or_404
from django.contrib.auth import get_user_model
from rest_framework.authtoken.models import Token

# User Registration (Signup) view
class RegisterView(APIView):
    def post(self, request):
        data = request.data
        User = get_user_model()
        user = User.objects.create_user(
            username=data['username'],
            email=data['email'],
            password=data['password'],
            first_name=data['first_name'],
            last_name=data['last_name']
        )
        serializer = UserSerializer(user)
        return Response(serializer.data, status=status.HTTP_201_CREATED)

# User Login view
class LoginView(APIView):
    def post(self, request):
        data = request.data
        user = get_user_model().objects.filter(email=data['email']).first()

        if user and user.check_password(data['password']):
            token, created = Token.objects.get_or_create(user=user)
            return Response({"token": token.key}, status=status.HTTP_200_OK)
        
        return Response({"detail": "Invalid credentials."}, status=status.HTTP_400_BAD_REQUEST)

# Blog List View
class BlogListView(APIView):
    def get(self, request):
        blogs = Blog.objects.all()
        serializer = BlogSerializer(blogs, many=True)
        return Response(serializer.data)

# Blog Detail View (View single blog)
class BlogDetailView(APIView):
    def get(self, request, pk):
        blog = get_object_or_404(Blog, pk=pk)
        serializer = BlogSerializer(blog)
        return Response(serializer.data)

# Blog Create View (Authenticated users only)
class BlogCreateView(APIView):
    permission_classes = [IsAuthenticated]

    def post(self, request):
        serializer = BlogSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(author=request.user)  # Assign the logged-in user as the author
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

# Blog Edit View (Authenticated users only and only the author can edit)
class BlogEditView(APIView):
    permission_classes = [IsAuthenticated]

    def put(self, request, pk):
        blog = get_object_or_404(Blog, pk=pk)
        if blog.author != request.user:
            return Response({'detail': 'You do not have permission to edit this blog.'}, status=status.HTTP_403_FORBIDDEN)

        serializer = BlogSerializer(blog, data=request.data, partial=True)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

8. **Create the URLs:**

Add the URLs for the registration, login, and blog-related views.

```python
# blog/urls.py
from django.urls import path
from .views import RegisterView, LoginView, BlogListView, BlogDetailView, BlogCreateView, BlogEditView

urlpatterns = [
    path('register/', RegisterView.as_view(), name='register'),
    path('login/', LoginView.as_view(), name='login'),
    path('blog/', BlogListView.as_view(), name='blog_list'),
    path('blog/create/', BlogCreateView.as_view(), name='create_blog'),
    path('blog/<int:pk>/', BlogDetailView.as_view(), name='view_blog'),
    path('blog/<int:pk>/edit/', BlogEditView.as_view(), name='edit_blog'),
]
```

9. **Include URLs in `blog_project/urls.py`:**

Include the blog app's URLs in the main project's URL configuration.

```python
# blog_project/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/', include('blog.urls')),  # Include blog app URLs
]
```

10. **Migrate the Database:**

Run migrations to apply the models.

```bash
python manage.py makemigrations
python manage.py migrate
```

11. **Install and Set Up Django REST Framework Authentication (Optional)**:

To use token authentication, you'll need to install `djangorestframework-authtoken` and configure it.

```bash
pip install djangorestframework-authtoken
```

Add `'rest_framework.authtoken'` to `INSTALLED_APPS` in `settings.py`:

```python
INSTALLED_APPS = [
    ...
    'rest_framework.authtoken',
]
```

Then run:

```bash
python manage.py migrate
```

12. **Run the Server:**

Finally, run the Django development server.

```bash
python manage.py runserver
```

### **Testing with Postman:**

1. **User Registration** (`POST /accounts/register/`):

   - **URL**: `http://127.0.0.1:8000/accounts/register/`
   - **Method**: POST
   - **Body** (JSON):

     ```json
     {
         "username": "john_doe",
         "email": "john@example.com",
         "password": "password123",
         "first_name": "John",
         "last_name": "Doe"
     }
     ```

2. **User Login** (`POST /accounts/login/`):

   - **URL**: `http://127.0.0.1:8000/accounts/login/`
   - **Method**: POST
   - **Body** (JSON):

     ```json
     {
         "email": "john@example.com",
         "password": "password123"
     }
     ```

   - **Response**:

     ```json
     {
         "token": "<your_generated_token_here>"
     }
     ```

3. **Create Blog** (`POST /blog/create/`):

   - **URL**: `http://127.0.0.1:8000/blog/create/`
   - **Method**: POST
   - **Headers**: `Authorization: Token <your_generated_token_here>`
   - **Body** (JSON):

     ```json
     {
         "title": "My First Blog",
         "content": "This is the content of my first blog."
     }
     ```

4. **List Blogs** (`GET /blog/`):

   - **URL**: `http://127.0.0.1:8000/blog/`
   - **Method**: GET
   - **Headers**: `Authorization: Token <your_generated_token_here>`

5. **View Blog** (`GET /blog/<pk>/`):

   - **URL**: `http://127.0.0.1:8000/blog/1/`
   - **Method**: GET
   - **Headers**: `Authorization: Token <your_generated_token_here>`

6. **Edit Blog** (`PUT /blog/<pk>/edit/`):

   - **URL**: `http://127.0.0.1:8000/blog/1/edit/`
   - **Method**: PUT
   - **Headers**: `Authorization: Token <your_generated_token_here>`
   - **Body** (JSON):

     ```json
     {
         "title": "Updated First Blog Title",
         "content": "This is the updated content of my first blog."
     }
     ```

---

This is a fully functional blog app using Django REST Framework, where users can register, log in, and create, view, and edit blogs.
