# 🔍 Django Search View 500 Error – Cause & Solution (FieldError on Content Search)

This README documents a common Django error related to search views that occur in production when using cPanel Python App Setup, especially with `DEBUG=False`.

---

## ❌ Issue Description

After deploying your Django project, visiting the `/search/` page with a query like:

```
https://yourdomain.com/search/?search=example
```

resulted in a **500 Internal Server Error**.

Because `DEBUG=False`, no detailed traceback was shown in the browser.

---

## 🔎 Root Cause

In your search view (`search_view`), you were trying to search the blog body using:

```python
Q(title__icontains=query) | Q(content__icontains=query)
```

But the `BlogModel` does **not** have a field named `content`.

Your actual field storing blog content is named:

```python
write_blogs = HTMLField()
```

Django throws a `FieldError` when trying to filter a non-existent model field.

---

## ✅ The Fix

1. Open your `views.py`
2. Locate the `search_view`
3. Update this line:

```python
Q(title__icontains=query) | Q(content__icontains=query)
```

✅ Replace with:

```python
Q(title__icontains=query) | Q(write_blogs__icontains=query)
```

Now Django can properly search using the actual field in your model.

---

## 🔁 Final Step: Restart Python App

After updating `views.py`, go to:

- cPanel → Setup Python App → Click **Restart**

This applies your changes in the production environment.

---

## 🧠 Best Practices to Avoid This Error

- ✅ Always double-check model field names using `BlogModel._meta.get_fields()`
- ✅ Log exceptions with `try/except` blocks in production views
- ✅ Use `python manage.py check` before deployment
- ✅ Monitor cPanel → Metrics → Errors for hidden server issues
- ✅ Test search view in `DEBUG=True` before switching to production

---

## 🏁 You're Done!

Your search view now correctly filters using the actual `write_blogs` field and the 500 error is resolved.

