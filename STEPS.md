# Login Testing Steps (`aggergrand-private`)

## 1) Open the repo
```powershell
cd C:\Users\Dom\OneDrive\Desktop\aggergrand-private
```

## 2) Ensure Python env works
Test the current venv:
```powershell
.\.venv\Scripts\python.exe --version
```

If that command errors, rebuild the venv:
```powershell
Remove-Item -Recurse -Force .\.venv
py -3.14 -m venv .venv
```

Activate and install backend deps:
```powershell
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install "django==6.0.3" "celery==5.6.2" "django-celery-results==2.6.0" "redis==6.4.0"
```

## 3) Set local env file
```powershell
Copy-Item .\backend\.env.example .\backend\.env -ErrorAction SilentlyContinue
```

In `backend/.env`, set:
```env
CELERY_TASK_ALWAYS_EAGER=1
```
This avoids needing Redis/Celery worker for login/register testing.

## 4) Initialize DB
```powershell
python .\backend\manage.py migrate
python .\backend\manage.py check
```

## 5) Run the app
```powershell
python .\backend\manage.py runserver 127.0.0.1:8000
```

Open:
`http://127.0.0.1:8000/login/login.html`

Important: open from `127.0.0.1:8000` (Django), not from the static workspace server.

## 6) Manual login test flow
1. Click `Create Account` with a valid email and password (8+ chars).
2. Confirm success message and redirect to onboarding.
3. Click `Log Out`, then sign back in from the login page.
4. Try wrong password and confirm the error.
5. Optional: click `Delete Account (Dev)` and re-register same email.

## 7) Optional API smoke test (PowerShell)
```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession

Invoke-RestMethod -Method Post -Uri "http://127.0.0.1:8000/api/auth/register" -WebSession $session -ContentType "application/json" -Body '{"email":"login.test@example.com","password":"Password123!","displayName":"Login Test"}'
Invoke-RestMethod -Method Get -Uri "http://127.0.0.1:8000/api/auth/me" -WebSession $session
Invoke-RestMethod -Method Post -Uri "http://127.0.0.1:8000/api/auth/logout" -WebSession $session
Invoke-RestMethod -Method Get -Uri "http://127.0.0.1:8000/api/auth/me" -WebSession $session
```

Expected final `/api/auth/me` response:
```json
{"user": null}
```
