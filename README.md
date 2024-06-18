# Django-Auto-SessionLogOut
To implement auto-logout functionality in our Django application, we need to adjust your session settings. This can be achieved by setting session expiration parameters and ensuring the middleware handles session timeouts.

## 1. Configure Session Expiry in Settings:
In `settings.py` file, add or update the following settings to control session expiration:
```python
# Time in seconds (e.g., 1800 seconds = 30 minutes)
SESSION_COOKIE_AGE = 1800  # Set your required time

# This setting ensures the session expires when the browser is closed
SESSION_EXPIRE_AT_BROWSER_CLOSE = True

# Use this setting if you want the session to expire after a period of inactivity
SESSION_SAVE_EVERY_REQUEST = True
```

## 2. Middleware Configuration:
```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware', # Message middleware
    'you_app_name.middleware.SessionTimeoutMiddleware',  # Custom middleware for session timeout warning
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

```
+ Firstly, ensure that `django.contrib.messages.middleware.MessageMiddleware` is included in your MIDDLEWARE setting in `settings.py`.
+ Ensure that `MessageMiddleware` is placed after SessionMiddleware and before other middleware that may interact with messages. Middleware order matters in Django, as it determines the sequence in which requests and responses are processed.
+ Make sure your `SessionTimeoutMiddleware` is placed after `SessionMiddleware` and before `AuthenticationMiddleware` in your MIDDLEWARE setting. 

## 3. Custom Middleware for Session Timeout:
Create a middleware file named `middleware.py` inside your Django app directory `app_name`:
```python
import time
from django.conf import settings

class SessionTimeoutMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        session_expiry_age = getattr(settings, 'SESSION_COOKIE_AGE', 1800)  # Default 30 minutes

        last_activity = request.session.get('last_activity')
        if last_activity:
            now = time.time()
            elapsed = now - last_activity
            if elapsed > session_expiry_age:
                request.session.flush()  # Clear the session data
                request.session['last_activity'] = now  # Update last activity timestamp
                return self.get_response(request)

        request.session['last_activity'] = time.time()  # Update last activity timestamp

        response = self.get_response(request)
        return response
```

## Middleware File Structure:

```python
project_directory/
    ├── project_directory/
    │   ├── __init__.py
    │   ├── settings.py
    │   ├── urls.py
    │   ├── wsgi.py
    ├── project_app/
    │   ├── __init__.py
    │   ├── models.py
    │   ├── views.py
    │   ├── middleware.py #This is middleware
    ├── manage.py

```
