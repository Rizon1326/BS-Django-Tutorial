
The **`AbstractBaseUser`** class in Django provides several built-in functionalities for user management, particularly around authentication. These built-in features help you manage user authentication, password hashing, and basic user-related operations without needing to implement them manually. Here's a breakdown of the key functionalities provided by **`AbstractBaseUser`**:

### 1. **Password Handling**:
   - **`set_password(raw_password)`**: This method allows you to securely set a user's password. It automatically hashes the password using Django's default password hashing algorithms. This means the raw password is not stored in the database, ensuring security.
   - **`check_password(raw_password)`**: This method is used to check if a provided password matches the stored hashed password. It is used when verifying a user's login credentials.
   - **`get_password_reset_token()`**: This method generates a token for password reset functionality, which can be sent to the user as part of the password reset process.

### 2. **Authentication Methods**:
   - **`is_authenticated`**: This is a boolean property (True by default) that checks whether the user is authenticated or not. It is essential for the authentication process in Django, indicating whether the user is logged in or not.
   - **`is_anonymous`**: This property (True by default) checks if the user is anonymous (i.e., not logged in).

### 3. **Date and Time Fields**:
   - **`date_joined`**: While not explicitly added in `AbstractBaseUser`, this is typically included in custom user models to track when the user was created. Django does not set this automatically, but it can be added by you when extending `AbstractBaseUser`.

### 4. **Basic Fields**:
   - **`is_active`**: This boolean field (True by default) is used to mark a user as active or inactive. An inactive user cannot log in. This is a commonly used feature for account deactivation in Django projects.
   - **`is_staff`**: This boolean field determines whether a user is a staff member with access to the Django admin panel. The default value is `False`, and you can set it to `True` to grant staff privileges.
   - **`is_superuser`**: This boolean field determines whether the user has superuser privileges, giving them access to all parts of the admin panel and unrestricted access to the system. 

### 5. **User Model Behaviors**:
   - **`USERNAME_FIELD`**: This is a required field that indicates the field used to identify users (by default, Django uses `username`, but here, since you’ve customized it, you would typically set it to `email` in your model). This makes sure the email address or any other field you choose is used as the primary identifier for logging in.

### 6. **Serialization and Querying**:
   - Since `AbstractBaseUser` is a subclass of `models.Model`, it provides all the basic ORM functionalities (such as `.save()`, `.delete()`, `.filter()`, etc.) that are available to all Django models, making it easy to query and interact with user records in the database.
   
### Why Use `AbstractBaseUser`:
- **Flexibility**: `AbstractBaseUser` allows you to define your own custom fields and authentication mechanisms. This is especially useful if you want to replace the default `username` field with something like `email` or add additional fields.
- **Built-in Security**: It provides built-in password hashing and secure password storage, so you don’t need to manually handle this important aspect of security.

In short, **`AbstractBaseUser`** gives you a foundational user model with essential authentication features, password management, and access control (via `is_active`, `is_staff`, `is_superuser`). You customize it by adding your own fields (such as `email`, `first_name`, etc.) and behavior to fit your application's needs.


### Custom User MOdel


### Step 1: Install Required Libraries

Before creating the custom user model, make sure you have installed Django and the necessary libraries for REST API:

```bash
pip install django
pip install djangorestframework
```

### Step 2: Set Up Your Django Project and App

First, create a Django project if you haven't done so already:

```bash
django-admin startproject myproject
cd myproject
```

Then create an app (e.g., `users`):

```bash
python manage.py startapp users
```

### Step 3: Create the Custom User Model

Now, we’ll create the custom user model. In the `users` app, create a `models.py` file (if it doesn't exist) and define the custom user model.

```python
# users/models.py

from django.db import models
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager
from django.utils import timezone

# Step 1: Custom Manager for User Model
class UserManager(BaseUserManager):
    def create_user(self, email, password=None, **extra_fields):
        """
        Create and return a user with an email, password, and other fields.
        """
        if not email:
            raise ValueError("The Email field must be set")
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)  # Set password using Django's built-in method
        user.save(using=self._db)
        return user

    def create_superuser(self, email, password=None, **extra_fields):
        """
        Create and return a superuser with an email, password, and other fields.
        """
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)
        return self.create_user(email, password, **extra_fields)


# Step 2: Define the Custom User Model
class User(AbstractBaseUser):
    email = models.EmailField(unique=True)
    first_name = models.CharField(max_length=255)
    last_name = models.CharField(max_length=255)
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)
    date_joined = models.DateTimeField(default=timezone.now)

    # Custom Manager
    objects = UserManager()

    USERNAME_FIELD = 'email'  # This will make email the username
    REQUIRED_FIELDS = ['first_name', 'last_name']  # Fields required during user creation

    def __str__(self):
        return self.email
```

### Explanation of the Code:

1. **UserManager**:
   - We create a custom manager to handle the creation of users and superusers. The `create_user` method ensures a password is hashed before saving the user to the database.
   - The `create_superuser` method is used to create superuser accounts with admin permissions.

2. **User Model**:
   - We define our custom `User` model by extending `AbstractBaseUser`.
   - The model includes fields like `email`, `first_name`, `last_name`, `is_active`, `is_staff`, and `date_joined`.
   - The `USERNAME_FIELD` is set to `email`, meaning email will be used for authentication instead of a traditional username.

### Step 4: Configure the Custom User Model in `settings.py`

Now, we need to tell Django that we're using the custom user model instead of the default one. Open `settings.py` and add this line:

```python
# settings.py

AUTH_USER_MODEL = 'users.User'  # This tells Django to use the custom User model from the 'users' app
```

### Step 5: Create and Apply Migrations

Once the custom user model is set up, you need to create and apply the migrations:

1. Run the following command to create the migrations for your new model:

```bash
python manage.py makemigrations users
```

2. Then, apply the migrations to update the database:

```bash
python manage.py migrate
```

### Step 6: Create a Superuser (Optional)

Now that the custom user model is set up and the migrations are applied, you can create a superuser for the admin panel:

```bash
python manage.py createsuperuser
```

You’ll be prompted to enter an email, username, and password for the superuser.

### Step 7: Verify the Custom User Model

Once everything is set up, run your Django server:

```bash
python manage.py runserver
```

You can now access the Django admin panel at `http://127.0.0.1:8000/admin` and log in with the superuser credentials. You should be able to see your custom `User` model in the admin interface.

### Conclusion

This completes the implementation of the **Custom User Model**. You now have a custom user model that uses email for authentication and includes fields like first name, last name, and password.

Let me know once you have completed this part, and I can guide you on the next step regarding **Email Verification**.
