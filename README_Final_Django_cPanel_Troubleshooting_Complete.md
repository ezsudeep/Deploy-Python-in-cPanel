# 🛠 Django + Tailwind (or Bootstrap/plain) Deployment on cPanel – Troubleshooting Guide

This is **comprehensive troubleshooting README** for Django (with Tailwind, Bootstrap, or plain CSS/JS) on cPanel using the Python Setup App. It includes **real issues**, **causes**, and **fixes**—based on deployment experience and community cases.

---

## ✅ What to Do If `.env` File Doesn’t Work

Sometimes `.env` doesn't load due to encoding, path issues, or misconfiguration.

### ✅ Fix 1: Use cPanel Environment Variables

Instead of relying solely on `.env`, define environment variables in cPanel:

**Steps:**

1. Go to **cPanel > Setup Python App > Edit**
2. Under “Environment Variables”, add:
   ```
   SECRET_KEY=your-secret-key
   DEBUG=False
   ALLOWED_HOSTS=yourdomain.com
   DATABASE_URL=mysql://user:pass@localhost/dbname
   ```

✅ These are picked up by Django via `os.environ` and are very reliable.

---

## 🧾 Example of ALLOWED_HOSTS in `settings.py`

```python
ALLOWED_HOSTS = [
    'localhost',
    '127.0.0.1',
    'yourdomain.com',
    'www.yourdomain.com',
    'ublogger5.codedbyujjal08.com'
]
```

---

## 🧩 Serving Static Files: `.htaccess` vs WhiteNoise

### ✅ Option 1: Use WhiteNoise (Recommended)

WhiteNoise lets Django serve static files directly via WSGI.

**Steps:**

1. Install:
   ```bash
   pip install whitenoise
   ```
2. In `settings.py`:
   ```python
   MIDDLEWARE.insert(1, 'whitenoise.middleware.WhiteNoiseMiddleware')
   STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
   ```
3. Run:
   ```bash
   python manage.py collectstatic --noinput
   ```

---

### 🔁 Option 2: Use Apache `.htaccess`

Use this if you can’t use WhiteNoise or prefer Apache-level handling:

```apache
RewriteEngine On
Alias /static/ /home/user/app/staticfiles/
<Directory /home/user/app/staticfiles>
    Require all granted
</Directory>
```

⚠️ If this fails, WhiteNoise is a safer fallback for cPanel.

---

## 🔧 Common Deployment Issues and Fixes

### 1. 🗃 Local SQLite → MySQL on cPanel

- Dump local data:
  ```bash
  python manage.py dumpdata > data.json
  ```
- Use:
  ```bash
  pip install mysql-connector-python
  ```
- Update `ENGINE`:
  ```python
  DATABASES['default']['ENGINE'] = 'mysql.connector.django'
  ```

---

### 2. ❌ `ModuleNotFoundError: MySQLdb`

**Fix:** Use `mysql-connector-python` instead of `mysqlclient`.

---

### 3. 🔑 `SECRET_KEY` not found

**Fix:** Use `.env` or define via cPanel > Edit Python App > Environment Variables.

---

### 4. ❌ Static CSS/JS not loading

- Run:
  ```bash
  python manage.py collectstatic --noinput
  ```
- Use WhiteNoise or configure `.htaccess`

---

### 5. 🧱 Static Not Found when `DEBUG=False`

Use WhiteNoise or Apache `.htaccess` as described above.

---

### 6. 🔒 SSL / HTTPS Not Working

Fix using cPanel > SSL/TLS > Run AutoSSL  
Force HTTPS with:

```apache
RewriteEngine On
RewriteCond %{SERVER_PORT} 80
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

---

### 7. 🛑 500 Internal Server Error

- Check cPanel > Metrics > Errors
- Run:
  ```bash
  python manage.py check
  ```
- Validate `passenger_wsgi.py` paths and ALLOWED_HOSTS

---

### 8. 🧷 404 on subpaths like `/search`, `/login`

Fix:
- Check URL patterns
- Restart app
- Check if template exists

---

### 9. ⚙️ Missing Packages in Production

Run:
```bash
pip freeze > requirements.txt
pip install -r requirements.txt
```

---

### 10. 🧭 File/Folder Permissions

- Folders: `755`
- Files: `644`
- Especially `.env`, staticfiles, templates

---

### 11. 🔄 Code Changes Not Reflecting

Always **restart** the app after:
- Updating views/templates
- Changing `.env`
- Modifying settings

---

### 12. 🧩 Activation Command Not Working in Terminal

Use the exact command from:
```
cPanel > Setup Python App > Edit
```

Example:
```bash
source /home/user/virtualenv/app/3.12/bin/activate && cd /home/user/app
```

---

### 13. 🐞 MIME Type Errors (CSS not applying)

- Ensure hashed files exist in `staticfiles/`
- Use WhiteNoise or .htaccess
- Clear browser cache

---

### 14. ⚠️ Passenger App Recognition Failure

- Ensure your project has a valid `passenger_wsgi.py`
- Confirm the folder exists
- Restart app even if not prompted

---

### 15. 🔍 Log Debugging

- cPanel > Metrics > Errors
- or check `/home/youruser/logs/error_log`

---

## 🧠 Best Practices Summary

- Use `.env` or cPanel env for secrets
- Use `collectstatic` on every update
- Restart the app manually after changes
- Monitor logs regularly
- Use WhiteNoise for static file delivery unless Apache routing is needed

---

## 🏁 Final Notes

With this guide, you can handle **90%+ of cPanel Django deployment errors** confidently.

**Need a PDF version, GitHub Wiki, or starter repo?** Just ask!
