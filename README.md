# extend-django-user-model
The corect way to extend Django User Model and use email and password to login


Most people cusotmizing or extending the Django user model in the oposit way,
I would like to share my experience how to do it the correct way without writing so much code.

# model.py
1- Here we extend **User** model by inherit **AbstractUser** class
2- Convert username field from models.**CharField** to **EmailField** and set  **unique=True**
3- Set **email** field to **None**
4- Set **REQUIRED_FIELDS** to empty array.
5- Set **EMAIL_FIELD** = "username"

<pre>
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
</pre>

In your **app**.admin.py in my case my app name is **users**

#H1 admin.py
We need to inherit **UserAdmin** class and finally register both **User** and **UserAdmin**

<pre>
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
</pre>


![](https://github.com/residentcode/extend-django-user-model/blob/main/create-superuser.png)
