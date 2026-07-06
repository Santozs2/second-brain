---
title: "Stack Final e Estrutura de Repositório — ChatBot WhatsApp"
aliases: ["Stack Final e Estrutura de Repositório — ChatBot WhatsApp", "Stack e Estrutura de Repositório"]
tags: [arquitetura, stack, repo, setup, chatbot]
status: fechado
projeto: ChatBot
criado: 2026-07-06
---

> [!info] Projeto: [[ChatBot|💬 ChatBot]]

# Stack e Estrutura de Repositório

## 🔧 Stack Final

### Backend
- **Framework:** Django 5.0+ com Django REST Framework
- **Banco:** PostgreSQL (Postgres 14+)
- **Cache/Real-time:** Redis + Django Channels (WebSocket)
- **Fila assíncrona:** Celery + Redis
- **Auth:** JWT (djangorestframework-simplejwt) + Session
- **CORS:** django-cors-headers
- **Docs:** drf-spectacular (Swagger/OpenAPI)
- **Validação:** Pydantic (via drf-pydantic)

### Frontend
- **Framework:** React 18+ com TypeScript
- **Styling:** Tailwind CSS + Shadcn/ui (componentes)
- **Editor de fluxos:** React Flow
- **HTTP:** axios
- **WebSocket:** socket.io-client (pra inbox em tempo real)
- **State:** Zustand ou Context API
- **Build:** Vite

### DevOps / Infra
- **Versionamento:** Git + GitHub
- **Hosting Backend:** Railway, Render, ou DigitalOcean App Platform
- **Hosting Frontend:** Vercel ou Netlify
- **Banco de dados:** PostgreSQL gerenciado (RDS, Supabase, ou Railway)
- **Redis:** Redis gerenciado (UpStash, Railway)
- **Secrets:** variáveis de ambiente (.env)

### IA (Fase 5+)
- **LLM:** Claude (Anthropic) ou GPT-4 (OpenAI)
- **RAG:** Qdrant (vector DB) ou Pinecone
- **Integração:** LangChain ou direto pela API

---

## 📁 Estrutura de Repositório

```
zapinbox/
│
├── backend/                    # Django + API
│   ├── config/
│   │   ├── __init__.py
│   │   ├── settings.py         # Configurações Django
│   │   ├── urls.py
│   │   ├── asgi.py             # Channels + WebSocket
│   │   ├── wsgi.py
│   │   ├── celery.py           # Celery config
│   │   └── __init__.py
│   │
│   ├── common/                 # Modelos e utilitários compartilhados
│   │   ├── models.py           # TenantModel, TimeStampedModel
│   │   ├── managers.py         # TenantManager, etc
│   │   ├── middleware.py       # TenantMiddleware
│   │   ├── decorators.py       # @require_tenant, etc
│   │   ├── serializers.py      # Serializers base
│   │   ├── permissions.py      # Custom permissions
│   │   └── __init__.py
│   │
│   ├── accounts/               # User + auth
│   │   ├── models.py           # User customizado
│   │   ├── managers.py         # UserManager
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   └── __init__.py
│   │
│   ├── organizations/          # Organization + Membership (multi-tenancy)
│   │   ├── models.py           # Organization, Membership
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   └── __init__.py
│   │
│   ├── whatsapp/               # Integração Cloud API da Meta
│   │   ├── models.py           # WhatsAppChannel
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── webhook.py          # Recebe callbacks da Meta
│   │   ├── client.py           # Client da Cloud API
│   │   ├── urls.py
│   │   ├── tasks.py            # Celery tasks
│   │   ├── admin.py
│   │   ├── apps.py
│   │   └── __init__.py
│   │
│   ├── contacts/               # Contatos (leads/clientes)
│   │   ├── models.py           # Contact, Tag
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── filters.py          # DjangoFilterBackend
│   │   ├── urls.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   └── __init__.py
│   │
│   ├── inbox/                  # Inbox multi-atendente (Fase 1)
│   │   ├── models.py           # Conversation, Message
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── consumers.py        # WebSocket consumers (Channels)
│   │   ├── routing.py          # WebSocket routing
│   │   ├── urls.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   └── __init__.py
│   │
│   ├── manage.py
│   ├── requirements.txt         # Dependências Python
│   ├── .env.example            # Variáveis de ambiente (exemplo)
│   ├── Dockerfile              # Imagem Docker (opcional, pra produção)
│   └── docker-compose.yml      # Orquestra containers (opcional)
│
├── frontend/                   # React + TypeScript + Vite
│   ├── src/
│   │   ├── components/
│   │   │   ├── Inbox/          # Componentes do inbox
│   │   │   │   ├── ConversationList.tsx
│   │   │   │   ├── MessageInput.tsx
│   │   │   │   └── ConversationDetail.tsx
│   │   │   ├── Layout/
│   │   │   │   ├── Sidebar.tsx
│   │   │   │   ├── Header.tsx
│   │   │   │   └── MainLayout.tsx
│   │   │   ├── Auth/
│   │   │   │   ├── Login.tsx
│   │   │   │   ├── Register.tsx
│   │   │   │   └── ProtectedRoute.tsx
│   │   │   ├── Common/
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Input.tsx
│   │   │   │   └── Modal.tsx
│   │   │   └── FlowBuilder/     # (Fase 2 — React Flow)
│   │   │       ├── FlowEditor.tsx
│   │   │       └── NodeTypes.tsx
│   │   │
│   │   ├── pages/
│   │   │   ├── Dashboard.tsx
│   │   │   ├── InboxPage.tsx
│   │   │   ├── SettingsPage.tsx
│   │   │   └── NotFound.tsx
│   │   │
│   │   ├── services/
│   │   │   ├── api.ts           # Axios instance + interceptors
│   │   │   ├── auth.ts          # Login/logout
│   │   │   ├── inbox.ts         # Chamadas de conversa/mensagem
│   │   │   ├── contacts.ts
│   │   │   └── websocket.ts     # Socket.io config
│   │   │
│   │   ├── hooks/
│   │   │   ├── useAuth.ts
│   │   │   ├── useInbox.ts
│   │   │   └── useWebSocket.ts
│   │   │
│   │   ├── store/               # Zustand ou Context
│   │   │   ├── authStore.ts
│   │   │   ├── inboxStore.ts
│   │   │   └── uiStore.ts
│   │   │
│   │   ├── types/
│   │   │   ├── index.ts         # Tipos globais
│   │   │   ├── api.ts
│   │   │   └── domain.ts
│   │   │
│   │   ├── styles/
│   │   │   ├── globals.css      # Tailwind + custom
│   │   │   └── variables.css    # CSS vars pro tema
│   │   │
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   └── index.html
│   │
│   ├── public/
│   │   ├── favicon.ico
│   │   └── logo.svg
│   │
│   ├── package.json
│   ├── vite.config.ts
│   ├── tsconfig.json
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   ├── .env.example
│   └── .gitignore
│
├── docs/                       # Documentação
│   ├── setup.md
│   ├── api.md                  # Endpoints da API
│   ├── architecture.md
│   └── deployment.md
│
├── .gitignore
├── README.md
├── LICENSE
└── .github/
    └── workflows/
        ├── backend-tests.yml   # CI/CD
        └── frontend-tests.yml
```

---

## 📝 Arquivos de Configuração

### `.gitignore`
```
# Python
__pycache__/
*.py[cod]
*$py.class
.venv/
env/
venv/
*.egg-info/
dist/

# Django
*.sqlite3
db.sqlite3
/staticfiles/
/media/

# Node
node_modules/
dist/
.next/
npm-debug.log

# Environment
.env
.env.local
.env.*.local

# IDEs
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db
```

### `backend/requirements.txt`
```
Django==5.1.0
djangorestframework==3.15.0
djangorestframework-simplejwt==5.3.0
django-cors-headers==4.3.0
django-environ==0.11.0
drf-spectacular==0.27.0
channels==4.1.0
channels-redis==4.2.0
celery==5.4.0
redis==5.0.0
psycopg[binary]==3.2.0
python-decouple==3.8
```

### `backend/.env.example`
```
# Django
DJANGO_SECRET_KEY=your-secret-key-here-change-in-production
DJANGO_DEBUG=True
DJANGO_ALLOWED_HOSTS=localhost,127.0.0.1

# Database
POSTGRES_DB=zapinbox
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your-password
POSTGRES_HOST=localhost
POSTGRES_PORT=5432

# Redis
REDIS_URL=redis://localhost:6379/0

# Channels
USE_REDIS_CHANNEL_LAYER=False

# Celery
CELERY_EAGER=False

# WhatsApp Cloud API
WHATSAPP_VERIFY_TOKEN=your-webhook-token
WHATSAPP_API_VERSION=v20.0

# Frontend URL
FRONTEND_URL=http://localhost:5173
```

### `frontend/.env.example`
```
VITE_API_URL=http://localhost:8000
VITE_WS_URL=ws://localhost:8000
```

---

## 🚀 Como rodar localmente

### Backend
```bash
cd backend

# Setup
python -m venv .venv
source .venv/bin/activate  # ou .venv\Scripts\activate no Windows
pip install -r requirements.txt

# Migrations
python manage.py makemigrations
python manage.py migrate

# Criar super user
python manage.py createsuperuser

# Rodar
python manage.py runserver

# Em outro terminal: Celery worker
celery -A config worker --loglevel=info
```

### Frontend
```bash
cd frontend

# Install
npm install

# Dev
npm run dev

# Build
npm run build
```

---

## 📊 Padrões de código

### Naming conventions

**Python/Django:**
- Classes: `PascalCase` (Contact, Conversation)
- Funções/métodos: `snake_case` (get_conversations, send_message)
- Constantes: `UPPER_SNAKE_CASE` (MAX_MESSAGE_LENGTH)
- Apps Django: `snake_case` (inbox, whatsapp, contacts)

**TypeScript/React:**
- Componentes: `PascalCase` (InboxList, MessageInput)
- Funções/hooks: `camelCase` (useInbox, sendMessage)
- Tipos: `PascalCase` (User, Conversation)
- Arquivos: `kebab-case` ou `PascalCase` (MessageInput.tsx)

---

## 🔄 Fluxo de desenvolvimento

1. **Feature branch:** `git checkout -b feature/inbox-websocket`
2. **Commit:** `git commit -m "feat: add websocket support to inbox"`
3. **Push:** `git push origin feature/inbox-websocket`
4. **Pull Request** → Review → Merge

---

## 📌 Resumo executivo

| Aspecto | Decisão |
|---------|---------|
| Backend | Django + DRF |
| Frontend | React + TypeScript + Vite |
| DB | PostgreSQL |
| Real-time | Django Channels + Redis |
| Fila | Celery + Redis |
| Hosting Backend | Railway/Render |
| Hosting Frontend | Vercel |
| Auth | JWT + Session |
| Docs API | drf-spectacular (Swagger) |

---

## 🎁 Próximos passos

- [ ] Criar repositório GitHub (zapinbox ou seu nome)
- [ ] Clonar e rodar localmente
- [ ] Configurar CI/CD (GitHub Actions)
- [ ] Registrar app na Meta for Developers
- [ ] Começar Fase 1 (models + auth)

---

## 📚 Referências

- [[Roadmap — Plataforma de Atendimento WhatsApp]]
- [[Multi-Tenancy no Django — Implementação]]
