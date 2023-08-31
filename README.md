# extend-django-user-model
The correct way to extend Django User Model and use email and password to login


Most people cusotmizing or extending the Django user model in the oposit way,
I would like to share my experience how to do it the correct way without writing so much code.

# model.py
1- Here we extend **User** model by inherit **AbstractUser** class<br>
2- Convert username field from models.**CharField** to **EmailField** and set  **unique=True**<br>
3- Set **email** field to **None**<br>
4- Set **REQUIRED_FIELDS** to empty array.<br>
5- Set **EMAIL_FIELD** = "username"<br>

```py
	from django.contrib.auth.models import AbstractUser
	from django.core.mail import EmailMessage
	from django.db import models
	from django.utils.translation import gettext_lazy as _
	
	
	class User(AbstractUser):
	username = models.EmailField(_('email'), max_length=255, unique=True)
	is_verified = models.BooleanField(default=False)
	is_banned = models.BooleanField(default=False)
	email = models.CharField(max_length=100, default=None)

	REQUIRED_FIELDS = []
	USERNAME_FIELD = "username"
	EMAIL_FIELD = "username"
	
	class Meta:
		verbose_name = _("user")
		verbose_name_plural = _("users")
	
	def email_user(self, subject, message, from_email=None, **kwargs) -> int:
		""" Email this user."""
		msg = EmailMessage(subject, message, from_email, [self.username], **kwargs)
	return msg.send()
	
	def __str__(self):
		return self.username
```
# settings.py
Set AUTH_USER_MODEL = 'users.User' replace users with your app name.
```py
AUTH_USER_MODEL = 'users.User'
```

In your **app**.admin.py in my case my app name is **users**

#H1 admin.py
We need to inherit **UserAdmin** class and finally register both **User** and **UserAdmin**

```py
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from django.utils.translation import gettext_lazy
from users.models import User


class UsersAdmin(UserAdmin):
    list_display = ("username",)
    fieldsets = (
        (gettext_lazy('Authentication'), {"fields": ("username", "password")}),
        (gettext_lazy("Personal info"), {"fields": ("first_name", "last_name")}),
        (
            gettext_lazy("Permissions"),
            {
                "fields": (
                    "is_active",
                    "is_staff",
                    "is_superuser",
                    "groups",
                    "user_permissions",
                ),
            },
        ),
        (gettext_lazy("Verifications"), {"fields": ("is_verified", "is_banned")}),
        (gettext_lazy("Important dates"), {"fields": ("last_login", "date_joined")}),
    )


admin.site.register(User, UsersAdmin)
```

From now on we are using Django built-in User model,<br>
We have added our extra fields and we can login/register by using email and password.<br>
We don't need to create custom UserManger().<br>
You can user reset-password built-in form in Django admin.<br>
We still need to use username field,<br>
Exp: 
```py
user = User.objects.filter(username='admin@admin.com').first()
user = User.objects.create_user(username='admin@admin.com', password='password')
```


![](https://github.com/residentcode/extend-django-user-model/blob/main/create-superuser.png)
![](https://github.com/residentcode/extend-django-user-model/blob/main/user_admin.png)

All done. <br>
I hope you enjoy coding.
