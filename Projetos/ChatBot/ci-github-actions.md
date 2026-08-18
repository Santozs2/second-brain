# CI — GitHub Actions + docker-compose

**Status:** ✅ COMPLETO
**Data:** 2026-07-21
**Commits:** `559b1c1` (pipeline + docker-compose) · `f58e11b` (badge no README)

## Visão Geral

CI = **Integração Contínua**. A cada `push` ou `pull_request` na `main`, o GitHub roda automaticamente uma bateria de checagens numa máquina limpa (runner Ubuntu). Se passa, aparece o selo verde **passing** no README; se quebra, chega e-mail e o PR fica marcado.

O objetivo aqui é **portfólio + rede de segurança**: qualquer pessoa (ou recrutador) que abrir o repo vê que o projeto compila, migra e passa no lint — sem precisar clonar nada.

## Decisão de honestidade (importante)

Todos os `tests.py` do backend estão **VAZIOS** (0 testes unitários — `grep -c "def test"` = 0). Os testes de verdade que existem são scripts E2E que exigem backend + frontend + banco rodando ao vivo (ver [[teste-infraestrutura]]), inviável num runner efêmero.

➡️ Um CI que rodasse `pytest backend/` daria **verde testando NADA** — pior que não ter CI, é um selo mentiroso.

**Solução:** CI honesto baseado em checagens que passam por motivos reais:

| Job | Passo | O que garante |
|-----|-------|---------------|
| backend | `manage.py check` | App carrega, sem erro de config/import |
| backend | `makemigrations --check --dry-run` | Migrations em dia (models = migrations) |
| backend | `migrate` (Postgres limpo) | Schema aplica do zero sem quebrar |
| frontend | `npm run lint` | ESLint sem erros |
| frontend | `npm run build` | `tsc -b` (checagem de tipos) + build Vite |

Suíte pytest versionada fica como próximo passo do roadmap ([[endurecimento-producao]]) — quando existir, é só trocar/adicionar o passo no job `backend` e o CI passa a testar de verdade.

## Arquivos Criados

```
chatbot/
├── .github/
│   └── workflows/
│       └── ci.yml            ← Pipeline (2 jobs: backend, frontend)
├── docker-compose.yml        ← Ambiente dev completo (1 comando)
└── backend/
    ├── Dockerfile            ← Imagem do backend
    └── .dockerignore         ← Exclui .venv, .env, __pycache__, db.sqlite3
```

## Pipeline (`.github/workflows/ci.yml`)

**Gatilhos:** `push` e `pull_request` na branch `main`.

### Job `backend` — checks + migrations
- Sobe um **service** Postgres 16 efêmero (com healthcheck `pg_isready`).
- `actions/setup-python@v5` (3.11) + cache de pip por `requirements.txt`.
- Instala deps → `manage.py check` → `makemigrations --check --dry-run` → `migrate`.
- Env do job: `DEBUG=False`, `POSTGRES_*` apontando pro service (`localhost:5432`), `WHATSAPP_WEBHOOK_VERIFY_TOKEN`, e uma **`FIELD_ENCRYPTION_KEY` Fernet descartável** (o app quebra no import sem uma chave válida).

### Job `frontend` — lint + build (typecheck)
- `actions/setup-node@v4` (Node 20 — Vite 7 exige) + cache npm por `package-lock.json`.
- `npm ci` → `npm run lint` → `npm run build`.

Os dois jobs rodam **em paralelo** (independentes).

## docker-compose (ambiente dev)

Um comando sobe tudo:

```bash
docker compose up --build
```

- **db:** postgres:16, volume `pgdata`, healthcheck.
- **redis:** redis:7 (necessário pro WebSocket cross-thread — ver limitação do InMemoryChannelLayer).
- **backend:** `build ./backend`, `depends_on` db (healthy) + redis (started), roda `migrate && runserver 0.0.0.0:8000`.

Valores de ambiente no compose são **só de DEV** (SECRET_KEY insegura, chave Fernet descartável) — não usar em produção. Backend em `http://localhost:8000`.

`backend/Dockerfile`: `python:3.11-slim`, `PYTHONDONTWRITEBYTECODE`, instala requirements, `CMD` migrate + runserver.

## Validação (feita antes do commit)

- `manage.py check` local → "0 issues".
- `makemigrations --check` → "No changes detected".
- `yaml.safe_load(ci.yml)` e `docker compose config` → parse OK.
- Chave Fernet do CI gerada com `Fernet.generate_key()` (descartável, não é segredo real).

## Badge no README

```markdown
[![CI](https://github.com/Santozs2/chatbot/actions/workflows/ci.yml/badge.svg)](https://github.com/Santozs2/chatbot/actions/workflows/ci.yml)
```

## Próximas Melhorias

1. **Suíte pytest versionada** → transforma o CI de "checagens" em "testes de verdade" (isolamento multi-tenant, webhook HMAC, motor de fluxo).
2. **Coverage** (pytest-cov) com badge de %.
3. **Cache de camadas Docker** no CI pra acelerar builds.
4. **Deploy automático** (CD) num ambiente de staging quando a main passa.

---

**Conclusão:** CI honesto no ar. Verde = app carrega, migra do zero e front compila/lint. O passo pra evoluir é a suíte de testes automatizados — aí o mesmo pipeline vira uma rede de segurança real.

Links: [[teste-infraestrutura]] · [[endurecimento-producao]] · [[stack-e-estrutura-repo]]
