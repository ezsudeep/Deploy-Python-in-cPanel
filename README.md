# How to Deploy a Django App with Tailwind CSS on cPanel using the Python Setup App

This guide provides a **step-by-step walkthrough** to deploy a Django project (with Tailwind CSS) on a cPanel-based hosting environment using the **Python App Setup tool**.

---

## ✅ Requirements

### Tools Needed:
- Access to a hosting provider with **cPanel** that supports **Python App Setup** (e.g., ProtozoaHost)
- Your **Django + Tailwind** project (e.g., from GitHub)
- A local development environment with Python and Node.js (for Tailwind build)
- Basic understanding of terminal commands

### Django Assumptions:
- Django version: 3.x to 5.x
- Tailwind CSS is compiled locally (cPanel does not support Node.js)

---

## 🔧 Step 1: Prepare Your Django Project Locally

### 1.1 Update `settings.py`
Ensure your `settings.py` has:
```python
import os
from pathlib import Path
BASE_DIR = Path(__file__).resolve().parent.parent
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

Use `django-environ` for `.env`:
```python
import environ
env = environ.Env()
environ.Env.read_env(os.path.join(BASE_DIR, '.env'))
```

### 1.2 Create a `.env` file
```
SECRET_KEY=your-secret-key
DEBUG=False
ALLOWED_HOSTS=yourdomain.com
DATABASE_URL=mysql://user:password@localhost/dbname
```

### 1.3 Compile Tailwind CSS
```bash
npm install
npx tailwindcss -i ./input.css -o ./output.css --minify
```
Put the final `output.css` into your `static/` directory.

### 1.4 Collect Static Files
```bash
python manage.py collectstatic --noinput
```

### 1.5 Freeze Requirements
```bash
pip freeze > requirements.txt
```

---

## 📂 Step 2: Zip and Upload to cPanel

### 2.1 Create a ZIP
Zip your project (excluding `venv/`, `.git/`, `node_modules/`).

### 2.2 Upload to File Manager
1. Go to **File Manager** in cPanel
2. Navigate to `/home/youruser/project-root/`
3. Upload and extract your project
4. Ensure `manage.py` is in the root of the folder

---

## 📃 Step 3: Create Python App in cPanel

1. Open **Setup Python App** in cPanel
2. Click **Create Application**
    - Python version: 3.12+
    - Application root: `blog-application` (or your folder name)
    - Application startup file: `passenger_wsgi.py`
    - Application entry point: `application`

> When you first create the Python App, cPanel **auto-generates** folders such as `public/`, `tmp/`, and a placeholder `passenger_wsgi.py`. You can delete those later or leave them, but replace the `passenger_wsgi.py` with your own.

---

## 🛠️ Step 4: Create `passenger_wsgi.py`

Inside your app root, create `passenger_wsgi.py`:
```python
import sys
import os
project_home = '/home/youruser/blog-application'
if project_home not in sys.path:
    sys.path.insert(0, project_home)
os.environ['DJANGO_SETTINGS_MODULE'] = 'yourproject.settings'
from django.core.wsgi import get_wsgi_application
application = get_wsgi_application()
```

---

## 📄 Step 5: Install Dependencies & Migrate DB

### 5.1 Open Terminal in cPanel:
```bash
source /home/youruser/virtualenv/blog-application/3.12/bin/activate
cd /home/youruser/blog-application
```

### 5.2 Install packages:
```bash
pip install -r requirements.txt
```

If using MySQL:
```bash
pip install mysql-connector-python
```

> ❌ **Common Error**: `ModuleNotFoundError: No module named 'MySQLdb'`
> - Fix: Use `mysql-connector-python` and change your ENGINE to `mysql.connector.django`

### 5.3 Run migrations:
```bash
python manage.py migrate
```

> ❌ **Possible Error**: `ImproperlyConfigured: Set the SECRET_KEY environment variable`
> - Fix: Ensure `.env` file is in your root directory and readable by Django

### 5.4 Create superuser (optional):
```bash
python manage.py createsuperuser
```

### 5.5 Collect static (again if needed):
```bash
python manage.py collectstatic --noinput
```

---

## 🌐 Step 6: Configure Static Files with WhiteNoise [https://whitenoise.readthedocs.io/en/latest/]

### 6.1 Install WhiteNoise
```bash
pip install whitenoise
```

### 6.2 Update `settings.py`
```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',
    ...
]
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
```

### 6.3 Restart App from cPanel UI
Go to **Python App Setup** > Click **RESTART**.

---

## ✅ Final Result

Visit: `https://yourdomain.com`

- If you see the Django site styled properly — ✅ Success!
- If static files still fail, ensure `.env`, static file path, and WhiteNoise config are in place

---

## 🔧 Troubleshooting

| Problem                      | Fix                                                                 |
|-----------------------------|----------------------------------------------------------------------|
| 500 Internal Server Error   | Check error logs in cPanel Metrics or terminal logs                  |
| CSS Not Loading             | Use WhiteNoise middleware and `collectstatic` output                 |
| `SECRET_KEY` error          | Ensure `.env` file exists and is properly formatted                  |
| `MySQLdb` error             | Use `mysql-connector-python` and update `ENGINE`                    |
| `ModuleNotFoundError`       | `pip install` missing package                                        |
| 404 on `/static/` files     | Use WhiteNoise or check Apache `.htaccess` (last resort)            |

---

## 🏑 You're Done!

Your Django app with Tailwind CSS is now **live on cPanel**, fully backed by a MySQL database, secured `.env` config, and served static assets reliably via WhiteNoise.

> If you need bonus features like email, domain redirects etc. — build from here!


## Does cPanel support Python, Java, or other frameworks?
https://support.cpanel.net/hc/en-us/articles/4408475507479-Does-cPanel-support-Python-Java-or-other-frameworks
