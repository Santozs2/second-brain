# Sprint 1 — Fundação (backend)

Issues: #2, #3, #4, #5 — [[kickoff]]

## Issue #2 — feat: setup do projeto Django (venv, apps accounts + finance)

**Estado ao começar**: `django-admin startproject` já rodado (pasta `backend/backend/`), apps `accounts` e `finance` já criados via `startapp` (ainda vazios, sem models). Falta: venv, dependências, config via env, User customizado, endpoint de login.

**Ordem sugerida (do mais simples pro mais completo):**

1. **venv + dependências** — criar `backend/.venv`, instalar `django`, `djangorestframework`, `djangorestframework-simplejwt`, `django-environ`, `django-cors-headers`, `psycopg2-binary`. Gerar `backend/requirements.txt` (`pip freeze`).
2. **`.env` + `.env.example`** — variáveis: `SECRET_KEY`, `DEBUG`, `ALLOWED_HOSTS`, dados do Postgres (`DB_NAME`/`DB_USER`/`DB_PASSWORD`/`DB_HOST`/`DB_PORT`). `.env` no `.gitignore` (já está), só o `.example` vai pro Git.
3. **`settings.py` via env** — trocar os valores hardcoded por `env(...)` (lib `django-environ`), trocar `DATABASES` de SQLite pra Postgres, registrar `rest_framework` + `corsheaders` + `accounts` + `finance` em `INSTALLED_APPS`, configurar `SIMPLE_JWT` básico (tempo de vida do access/refresh token).
4. **User customizado (`accounts/models.py`)** — herdar de `AbstractUser`, login por **email** em vez de username (`USERNAME_FIELD = "email"`), manager customizado sem exigir username. **Ponto crítico**: setar `AUTH_USER_MODEL = "accounts.User"` no settings ANTES da primeira `migrate` — trocar depois de já ter migrations aplicadas é bem mais trabalhoso.
5. **Endpoint de login** — `TokenObtainPairView` do simplejwt (ou serializer próprio se quiser devolver dados do usuário junto), rotas `/api/auth/login/` e `/api/auth/refresh/`.

**DoD**: `manage.py check` limpo, `makemigrations`/`migrate` rodando sem erro, `POST /api/auth/login/` devolvendo `access`/`refresh` pra um usuário criado via `createsuperuser`.

**Observação pega em `settings.py` atual**: `TIME_ZONE = 'UTF-8'` está errado (isso é encoding, não timezone) — corrigir pra algo como `'America/Sao_Paulo'` já que estamos reescrevendo esse arquivo nessa issue.
