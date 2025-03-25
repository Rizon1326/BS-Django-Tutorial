
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

# BaseUserManager

The **`BaseUserManager`** class in Django provides basic management functionality for creating, managing, and handling user-related operations, particularly in the context of a custom user model. Here are the basic functionalities that **`BaseUserManager`** provides:

### 1. **User Creation Methods**:
   - **`create_user(email, password=None, **extra_fields)`**:
     - This method is used to create a regular user. It is responsible for:
       - Normalizing the email (using `normalize_email`).
       - Setting the password securely (using `set_password`).
       - Saving the user instance to the database.
     - **Purpose**: To create a standard user with a password and any additional fields passed through `extra_fields`.
   
   - **`create_superuser(email, password=None, **extra_fields)`**:
     - This method is used to create a superuser. It is a specialized version of `create_user`, but it ensures that the created user has both the `is_staff` and `is_superuser` flags set to `True`.
     - **Purpose**: To create a user with superuser privileges (i.e., access to the Django admin panel and full control over the application).

### 2. **Normalizing Email**:
   - **`normalize_email(email)`**:
     - This method normalizes the email address by converting it to lowercase (this is important because emails are typically case-insensitive).
     - **Purpose**: Ensures consistency when storing and comparing email addresses in the database.

### 3. **Custom Manager Setup**:
   - The **`BaseUserManager`** class is intended to be extended to create a custom user manager for your user model. By creating a custom manager (like `UserManager` in your code), you can add custom user creation methods (e.g., `create_user`, `create_superuser`) and override default behaviors, but still retain the basic functionality like `normalize_email`.

### 4. **Saving User**:
   - **`user.save(using=self._db)`**: This is part of the `create_user` and `create_superuser` methods. After setting the password and populating the user fields, this function saves the user instance to the database.
   - **Purpose**: It ensures that the user is properly saved to the database after their details have been processed.

### Summary of Basic Functionalities:
1. **`create_user`**: Creates a normal user with the required fields (email, password, etc.).
2. **`create_superuser`**: Creates a superuser with admin privileges.
3. **`normalize_email`**: Normalizes the email address for consistency and storage.
4. **Saving**: Handles saving the user instance to the database.

### Why Use **`BaseUserManager`**:
- It provides the foundation for user creation, password handling, and normalization of the email field.
- It simplifies the process of creating and managing users (especially superusers).
- It allows you to extend and customize the behavior of user creation to fit the specific needs of your application (e.g., using email as the unique identifier). 


# Code for Custom User Model

### Create the Custom User Model

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
```
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
                                                     ### Explanation
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Explanation of the Code:
create_user Method:

Purpose: This method is used to create a normal user (not a superuser). It takes email, password, and any other extra fields (extra_fields) as arguments.

Key Actions:

Check if email is provided:

if not email: — This ensures that the email is required for creating a user. If no email is provided, it raises a ValueError.

Normalize the email:

email = self.normalize_email(email) — This normalizes the email address to ensure it is stored in a consistent format (e.g., all lowercase). Django’s normalize_email method helps with consistency when comparing email addresses.

Create the user:

user = self.model(email=email, **extra_fields) — This creates an instance of the User model (which is the custom user model in your code). It initializes the user with the normalized email and any other extra fields that are passed in.

Set the password:

user.set_password(password) — This method is used to hash and securely store the user’s password. The password is never stored in plain text.

Save the user:

user.save(using=self._db) — This saves the user instance to the database. The using=self._db part ensures that the user is saved to the appropriate database (useful when dealing with multiple databases).

create_superuser Method:

Purpose: This method is used to create a superuser, who will have special privileges in the Django admin panel and full access to the application.

Key Actions:

Set default fields:

extra_fields.setdefault('is_staff', True) — This ensures that the superuser will have is_staff set to True (this is a requirement for access to the Django admin panel).

extra_fields.setdefault('is_superuser', True) — This ensures that the superuser will have is_superuser set to True (this grants the user full admin privileges).

Create the superuser:

return self.create_user(email, password, **extra_fields) — Instead of duplicating the logic for creating a superuser, this method calls create_user (the previously defined method). It passes the email, password, and any extra fields (which now include is_staff and is_superuser), thus reusing the user creation logic.


---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
                                                     ### Explanation
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

E

```
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

Sure! Let's break down **Step 2**: Defining the **Custom User Model** and explain the code simply:

### Code Explanation:

1. **Inheriting from `AbstractBaseUser`**:
   - **`class User(AbstractBaseUser):`**
     - This line creates a custom user model by inheriting from `AbstractBaseUser`, which is a built-in Django class. By doing this, you're customizing the user model to meet your specific needs, like using **email** as the primary user identifier instead of **username**.

2. **Fields of the Custom User Model**:
   - **`email = models.EmailField(unique=True)`**:
     - This field is for storing the user's email address. The `unique=True` part ensures that no two users can have the same email in the database.
   - **`first_name = models.CharField(max_length=255)`**:
     - This field is for storing the user's first name.
   - **`last_name = models.CharField(max_length=255)`**:
     - This field is for storing the user's last name.
   - **`is_active = models.BooleanField(default=True)`**:
     - This field indicates if the user account is active. By default, it is set to `True` (meaning the user can log in).
   - **`is_staff = models.BooleanField(default=False)`**:
     - This field indicates whether the user is a staff member (able to access the Django admin panel). It is `False` by default.
   - **`date_joined = models.DateTimeField(default=timezone.now)`**:
     - This field records the date and time when the user joined (created the account). By default, it is set to the current date and time.

3. **Custom Manager**:
   - **`objects = UserManager()`**:
     - This line assigns the custom manager `UserManager` to the `objects` attribute of the model. It allows you to use the `create_user()` and `create_superuser()` methods defined in `UserManager` for creating regular users and superusers.

4. **`USERNAME_FIELD`**:
   - **`USERNAME_FIELD = 'email'`**:
     - This line specifies that **email** will be used as the **username** for the user model. Normally, Django uses `username` as the login field, but here you are making email the unique identifier for logging in.

5. **`REQUIRED_FIELDS`**:
   - **`REQUIRED_FIELDS = ['first_name', 'last_name']`**:
     - This list defines the fields that are required during **user creation**. In this case, when you create a user, you must provide **first name** and **last name** in addition to the **email**. 
     - **Note**: `email` is not listed here because it's used as the `USERNAME_FIELD`, so Django automatically considers it a required field during user creation.

6. **String Representation**:
   - **`def __str__(self): return self.email`**:
     - This method defines how the user object will be displayed when you print it or look at it in the Django admin panel. In this case, it will display the user's email.

### Explanation of the Code:

1. **UserManager**:
   - We create a custom manager to handle the creation of users and superusers. The `create_user` method ensures a password is hashed before saving the user to the database.
   - The `create_superuser` method is used to create superuser accounts with admin permissions.

2. **User Model**:
   - We define our custom `User` model by extending `AbstractBaseUser`.
   - The model includes fields like `email`, `first_name`, `last_name`, `is_active`, `is_staff`, and `date_joined`.
   - The `USERNAME_FIELD` is set to `email`, meaning email will be used for authentication instead of a traditional username.

