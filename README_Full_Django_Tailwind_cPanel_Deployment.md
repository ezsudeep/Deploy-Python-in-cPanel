# 🛠️ Django + Tailwind CSS Deployment on cPanel using Python App — Full Guide + Troubleshooting

This is the **complete, battle-tested guide** to deploying Django projects with Tailwind CSS (or Bootstrap/plain CSS/JS) using cPanel’s Python App Setup tool. Includes real errors, fixes, and deployment strategies from actual deployments.

---

## ✅ Requirements

- Hosting with **cPanel** and **Python App Setup**
- Your Django + Tailwind project (zip or GitHub)
- MySQL database (cPanel-compatible)
- Node.js locally to compile Tailwind CSS
- Terminal knowledge (basic)

---

## 🔧 Step 1: Prepare Django Project Locally

### 1.1 Update `settings.py`

```python
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

### 1.2 Configure `.env`

Use `django-environ`:

```python
import environ
env = environ.Env()
environ.Env.read_env(os.path.join(BASE_DIR, '.env'))
```

Example `.env`:

```
SECRET_KEY=your-secret
DEBUG=False
ALLOWED_HOSTS=ublogger5.codedbyujjal08.com
DATABASE_URL=mysql://cpaneluser_dbuser:password@localhost/cpaneluser_db
```

### 1.3 Compile Tailwind CSS (locally)

```bash
npx tailwindcss -i ./input.css -o ./static/css/output.css --minify
```

### 1.4 Run

```bash
python manage.py collectstatic --noinput
pip freeze > requirements.txt
```

---

## 📦 Step 2: Upload to cPanel

1. Zip project (exclude `venv`, `.git`, `node_modules`)
2. Upload in **File Manager**
3. Extract in `/home/user/project-root/`
4. Make sure `manage.py` is in root

---

## 🐍 Step 3: Setup Python App in cPanel

1. Go to **Setup Python App**
2. Click **Create Application**
   - Python: 3.12+
   - Root: e.g. `blog-application`
   - Startup file: `passenger_wsgi.py`
   - Entry: `application`

> cPanel will auto-generate folders and files (e.g., `public/`, `tmp/`, default `passenger_wsgi.py`)

---

## 🚦 Step 4: Configure `passenger_wsgi.py`

```python
import sys, os
project_home = '/home/user/blog-application'
sys.path.insert(0, project_home)
os.environ['DJANGO_SETTINGS_MODULE'] = 'yourproject.settings'
from django.core.wsgi import get_wsgi_application
application = get_wsgi_application()
```

---

## 📄 Step 5: Terminal Setup

Activate virtualenv and install:

```bash
source /home/user/virtualenv/blog-application/3.12/bin/activate
cd /home/user/blog-application

pip install -r requirements.txt
pip install mysql-connector-python
python manage.py migrate
python manage.py collectstatic --noinput
```

---

## 🔧 Step 6: Fix Common Issues + Configs

### ❌ .env Not Working?

✅ Set variables manually in cPanel:
```
SECRET_KEY, DEBUG, ALLOWED_HOSTS, DATABASE_URL
```

### 🛠 `ALLOWED_HOSTS` Example

```python
ALLOWED_HOSTS = ['localhost', 'ublogger5.codedbyujjal08.com', 'yourdomain.com']
```

---

## 🧩 Static Files — Two Solutions

### ✅ Option 1: WhiteNoise (Recommended)

```bash
pip install whitenoise
```

```python
MIDDLEWARE.insert(1, 'whitenoise.middleware.WhiteNoiseMiddleware')
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
```

### 🔁 Option 2: Apache `.htaccess`

```apache
RewriteEngine On
Alias /static/ /home/user/blog-application/staticfiles/
<Directory /home/user/blog-application/staticfiles>
    Require all granted
</Directory>
```

---

## 🧪 Troubleshooting

| Problem                        | Fix                                                              |
|-------------------------------|-------------------------------------------------------------------|
| `.env` not loading            | Add in cPanel env or fix path using `environ.Env.read_env()`     |
| MySQLdb error                 | Use `mysql-connector-python` + `ENGINE = 'mysql.connector.django'` |
| `SECRET_KEY` error            | Ensure `.env` exists and has correct key                         |
| Static/CSS not loading        | Use WhiteNoise + collectstatic                                   |
| 404 on static paths           | Use Apache alias or WhiteNoise                                   |
| 500 Internal Server Error     | Restart app, check logs, validate `.env`                         |
| Terminal: python not found    | Use source command from Setup Python App > Edit                  |
| Files not applying            | Restart app manually from cPanel UI                              |
| SSL issues                    | Use AutoSSL in SSL/TLS section of cPanel                         |

---

## 🔁 Post-Deployment Best Practices

- Always restart app after `.env` or code updates
- Keep `requirements.txt` updated
- Use `collectstatic` after every CSS/JS change
- Use `.env` or cPanel env to avoid hardcoded secrets

---

## 🏁 You’re Fully Deployed!

Your Django + Tailwind app is now live, with proper static handling, MySQL backend, and scalable deployment practices on cPanel.

🔗 Need help automating deployment, configuring cron jobs, or sending emails via SMTP? Just ask.

