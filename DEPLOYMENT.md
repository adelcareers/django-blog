# Django Deployment Checklist (Heroku)

Use this checklist before each production deploy.

## 1) Local environment sanity

- Activate virtualenv:
  - `source .venv/bin/activate`
- Install dependencies:
  - `python -m pip install -r requirements.txt`
- Run checks:
  - `python manage.py check`
- Run migrations locally:
  - `python manage.py migrate`

## 2) Static files setup

- Confirm settings include:
  - `whitenoise.middleware.WhiteNoiseMiddleware` directly after `SecurityMiddleware`
  - `STATIC_URL = "/static/"`
  - `STATIC_ROOT = BASE_DIR / "staticfiles"`
  - `STORAGES["staticfiles"]["BACKEND"] = "whitenoise.storage.CompressedManifestStaticFilesStorage"`
- Build static files:
  - `python manage.py collectstatic --noinput`

## 3) Required Heroku config vars

Set these in Heroku app config:

- `SECRET_KEY` = strong random key
- `DATABASE_URL` = Postgres connection string
- `DEBUG` = `False`

Optional:

- `DISABLE_COLLECTSTATIC` should be unset (or not equal to `1`)

## 4) Allowed hosts and CSRF

- `ALLOWED_HOSTS` should include:
  - `.herokuapp.com`
- `CSRF_TRUSTED_ORIGINS` should include:
  - `https://*.herokuapp.com`

## 5) Deploy

- Commit and push:
  - `git add .`
  - `git commit -m "Deploy prep"`
  - `git push heroku main`

## 6) Post-deploy verification

- Run migrations on Heroku (if needed):
  - `heroku run python manage.py migrate -a <your-app-name>`
- Verify app:
  - Open `/admin/` and confirm styles load
  - Open browser dev tools Network tab and confirm `/static/admin/...` returns HTTP `200`

## 7) Quick troubleshooting

- Admin page has no styles:
  - Run `python manage.py collectstatic --noinput`
  - Confirm WhiteNoise middleware is present and correctly ordered
  - Confirm `DEBUG=False` in production
- `ModuleNotFoundError: No module named 'whitenoise'`:
  - Add `whitenoise==6.12.0` to `requirements.txt`
  - Redeploy
- Static 404 in production:
  - Re-check `STATIC_ROOT`, `STORAGES`, and that collectstatic ran during deploy
