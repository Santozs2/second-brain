---
tipo: contexto-tecnico
projeto: ChatBot
fonte: memoria persistente do Claude Code
snapshot: 2026-07-15
tags: [chatbot, whatsapp, django, react, contexto]
---

> [!info] O que e isso
> Snapshot da memoria tecnica que o Claude Code mantem sobre este projeto (arquivo `MEMORY.md`). Serve como referencia rapida do estado do codigo, decisoes e pegadinhas. Pode ficar desatualizado — a fonte viva fica na memoria do agente.

# Projeto ChatBot — Plataforma de Atendimento WhatsApp

## Comunicacao
- Falar em **portugues** com o usuario.

## Permissoes de workflow
- Autorizado a criar commits no repo sempre que achar necessario (permissao duravel).
- NAO commitar arquivos `.pyc` novos; adicionar arquivos por nome (evitar `git add -A`). `.pyc`/`__pycache__` e `db.sqlite3` JA removidos do tracking e no `.gitignore`.
- Estilo de commit: prefixo em portugues (`feat:`, `fix:`), mensagem curta.
- Aplicar edicoes/fixes locais direto, SEM pedir permissao. So confirmar em acoes arriscadas/irreversiveis (push autorizado, mas ele costuma pedir "da push").
- NAO adicionar trailer `Co-Authored-By: Claude`. Autor = so o usuario (Luis Antonio dos Santos Nogueira <luis.senaie@gmail.com>).

## Estrutura
- Backend Django em `backend/` (venv `backend/.venv`, python `backend/.venv/Scripts/python.exe`).
- Apps: `accounts` (User por email + JWT), `organizations` (Organization + Membership owner/admin/agent), `common` (TimeStampedModel/TenantModel/TenantManager + TenantMiddleware), `whatsapp`, `contacts`, `inbox`.
- Multi-tenancy: `TenantMiddleware` resolve `request.tenant` via header `X-Organization` (slug) ou 1a membership. `TenantModel.objects.for_tenant(tenant)`.
- DB: PostgreSQL (Supabase) via env vars.
- CONFIG via env: `settings.py` le `SECRET_KEY`, `DEBUG` (default False), `ALLOWED_HOSTS`, `REDIS_HOST`/`REDIS_PORT`, `CORS_ALLOWED_ORIGINS` (so quando DEBUG=False; em dev `CORS_ALLOW_ALL_ORIGINS=DEBUG`). Tudo no `.env` local; `.env.example` documenta. SECRET_KEY de dev ja vazada no git — GERAR NOVA em prod. `requirements.txt` pinado.
- Frontend URLs via env: `lib/api.ts` exporta `API_URL`/`WS_URL` de `import.meta.env.VITE_API_URL`/`VITE_WS_URL` (fallback localhost).

## Roadmap
- Fase 1 = Inbox multi-atendente. Sprint 1 fundacao, Sprint 2 Cloud API (webhook + client), Sprint 3 WebSocket (Redis), Sprint 4 frontend React — todos FEITOS. Atribuicao de conversas FEITA.
- Fase 2 = construtor de chatbot (React Flow + motor de fluxo). SO planejada. Regra de ouro: lancar Fase 1 e validar com cliente real ANTES da Fase 2.

## Sprint 2 (Cloud API)
- `whatsapp/client.py`: `send_text_message(channel, to, text)` → POST Graph API, retorna wa_message_id.
- `whatsapp/webhook.py`: `WhatsAppWebhookView` (APIView, AllowAny, sem auth). GET valida hub.verify_token → hub.challenge; POST sempre 200 (loga excecoes).
- `whatsapp/services.py`: `handle_webhook_payload` resolve tenant pelo `metadata.phone_number_id` → WhatsAppChannel. Inbound cria/atualiza Contact/Conversation/Message (idempotencia por wa_message_id); statuses atualizam sem regredir (rank queued<sent<delivered<read/failed).
- Env: `WHATSAPP_API_VERSION`, `WHATSAPP_WEBHOOK_VERIFY_TOKEN` (`chatbot-dev-verify-9f3a1c7e`).

## Sprint 4 — Frontend
- Stack: Vite + React + TS + Tailwind v4. Pasta `frontend/`. Backend `localhost:8000`.
- `lib/api.ts`: `apiFetch(path, opts)` injeta Bearer + json, 401 → `clearTokens()` + reload. `lib/auth.ts`: getToken/saveTokens/clearTokens.
- `App.tsx`: portao de auth → `Login` ou `Inbox`. Login POST `/api/auth/login/` → `{access, refresh}`.
- Componentes: `Inbox` (sidebar+main, dono do selected), `ConversationList` (GET conversations), `MessagePanel` (GET messages + WebSocket `/ws/inbox/?token=` filtrando por conversation_id, cleanup ws.close()).
- Endpoints inbox: `ConversationView(ReadOnlyModelViewSet)` + `@action messages` (GET lista / POST envio). Serializer: contact_name (=contact.profile_name), contact_wa_id, assigned_to, last_inbound_at, is_within_service_window.
- **Modo de trabalho**: hands-on — usuario escreve o codigo, eu passo spec + reviso. EXCECAO front: "n sou muito de front" → em polimento/UI eu implemento direto.
- Polimento visual (commit `28d20ca`): avatares (cor deterministica por hash), header do contato, estados loading/erro/vazio, bolhas rounded-2xl.
- Envio pelo atendente (commit `ccfbd74`): `_send_message` valida texto (400), checa janela 24h (403), cria Message out, tenta send, sucesso→sent / falha→failed (logado), broadcast. Front: campo de envio no rodape com dedupe por id.

## Atribuicao de conversas (commit `4422a81`)
- Backend `inbox/views.py`: `@action(detail=False, get) members` lista memberships do tenant (`{id, name, role}`, name = user.full_name or user.email). `@action(detail=True, post) assign` body `{membership_id}` (null desatribui); valida membership do MESMO tenant (400 se invalido — filtro obrigatorio por seguranca). `get_queryset` le `?assigned=me|unassigned|<membership_id>`. Serializer: `assigned_to` (id da FK) + `assigned_to_name` (SerializerMethodField).
- Front: `types.ts` `assigned_to_name` + tipo `Member`. `Inbox` abas Todas/Minhas/Nao atribuidas (filter + reloadKey; handleAssigned). `ConversationList` refetch em [filter, reloadKey], badge do atendente. `MessagePanel` busca /members/, dropdown no header, POST /assign/ → onAssigned.
- Bugs recorrentes dele: esquece o ponto do meio `m.user.full_name` (escreve `m.user_full`, `m.full_name`); campo em update_fields (`update_at`→`updated_at`); SerializerMethodField exige `get_<campo>` exato; logica invertida em guard clause; definir 2x get_queryset.

## Meta WhatsApp — status (usuario MENOR DE IDADE, sem CNPJ)
- Verificacao da empresa (destrava envio pro BR, erro 130497) EXIGE CNPJ (MEI serve, mas exige 18+) — nao da com CPF. **Recebimento funciona 100% sem verificacao**; envio pro Brasil bloqueado.
- Reframe do produto: no SaaS multi-tenant, cada CLIENTE conecta o Business verificado DELE (com CNPJ dele) — o dev nao precisa de CNPJ proprio; envio real fica pra validacao com cliente. Alternativa p/ testar envio: familiar 18+ com MEI, ou enviar p/ pais nao-restrito.
- App Meta "Chat Bot": app_id 1601345211616626, business_id 872435268853572. Numero de teste: `+15551431391`, phone_number_id 1171868172685122, waba_id 1000726339497309. Canal DB org zz_teste, status=connected, id `0beaced1-33fb-48ff-95a1-34aa3b089fa2`. Destinatario verificado `+55 17 99196-2290` (wa_id 5517991962290, COM o 9). Token do painel TEMPORARIO — usuario cola novo, eu atualizo `WhatsAppChannel.access_token`.
- Tunel: cloudflared via Docker (`docker run -d --name chatbot-tunnel cloudflare/cloudflared:latest tunnel --url http://host.docker.internal:8000`). URL nova a cada restart → reconfigurar Callback no Meta (Etapa 2 → Configurar webhooks). Backend PRECISA rodar em `0.0.0.0:8000`. `WHATSAPP_API_VERSION=v25.0`.
- **PEGADINHA #1**: verificar webhook + assinar campo `messages` NAO basta. Precisa TAMBEM `POST /v25.0/{WABA_ID}/subscribed_apps` (Bearer token). Sem isso Meta recebe mas nao dispara webhook.
- **PEGADINHA #2**: numero de teste so entrega/recebe DEPOIS que o destinatario inicia a conversa (mandar do celular pro numero de teste primeiro).
- Envio bloqueado: status `failed` code `130497` "Business account is restricted from messaging users in this country".
- `_process_statuses` loga `errors` do status → foi o que revelou o 130497.

## access_token criptografado (commit `d27e870`)
- `WhatsAppChannel.access_token` agora e `EncryptedTextField` (lib `encrypted_model_fields`). `INSTALLED_APPS` += `encrypted_model_fields`; `settings.FIELD_ENCRYPTION_KEY = env("FIELD_ENCRYPTION_KEY")` (chave Fernet no `.env`, placeholder no `.env.example`). Migration `whatsapp/0002` (AlterField, no-op de schema — DB continua text).
- **PEGADINHA da migracao**: token antigo estava em TEXTO PURO no banco → depois do campo virar encrypted, leitura via ORM tenta descriptografar e QUEBRA. FIX: re-salvar o token 1x pra gravar cifrado. Ler o valor cru via SQL (`SELECT access_token`), NAO via `.get()` (que ja descriptografa). Re-salvar com instancia vazia da INSERT (`_state.adding=True`) → NotNull em organization_id; forcar UPDATE: `ch=WhatsAppChannel(pk=pk); ch._state.adding=False; ch._state.db='default'; ch.access_token=plain; ch.save(update_fields=['access_token'])`. Cifrado no banco comeca com `gAAAAA` (prefixo Fernet).
- Gerar chave Fernet: `python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"`.
- Teste de seguranca do banco (validado): (1) valor cru = cifra `gAAAAA...`, sem pedaco do token real; (2) nenhuma coluna text/varchar do schema `public` contem o token em texto puro; (3) chave errada → `InvalidToken` (ilegivel). A chave e a joia: nunca no git, secret manager em prod. Rotacao exige re-salvar todos os tokens (lib suporta lista de chaves: 1a cifra, resto so descriptografa).

## Pendencias conhecidas
- Trocar token temporario por permanente via System User.

## Dicas de ambiente
- `.env` no `.gitignore` — nunca commitar.
- `pip` puro instala no Python GLOBAL errado! SEMPRE no venv: `./.venv/Scripts/python.exe -m pip install <pkg>` (de `backend/`).
- WebSocket: `channels` 4.3.2 + `daphne` 4.2.2. Grupo por org `org_{id}`, auth JWT no handshake (`?token=`, `common/ws_auth.py`).
- `InMemoryChannelLayer` NAO entrega broadcast webhook→WS num servidor real (thread/loop diferente). FIX: `channels_redis` + Redis.
- Redis via Docker: `docker run -d --name chatbot-redis -p 6379:6379 redis:7`. Hosts `[("127.0.0.1", 6379)]`.
- BUG DE VERSAO: `redis-py` 8.x incompativel com `channels_redis` 4.3.0 → WS cai em ~10s (close 1011). FIX: fixar `redis<6` (5.3.1).
- Credenciais dev: `teste@zz.com` / `teste1234` (org ZZ_TESTE, id 99e1a1db-...). Token rapido: `AccessToken.for_user(u)`.
- Django one-off: `python -c "import django,os; os.environ.setdefault('DJANGO_SETTINGS_MODULE','chatbot.settings'); django.setup(); ..."`. `manage.py shell` via heredoc quebra com blocos if/for no nivel 0.

## Links
- [[guia-fase-1-inbox]]
- [[retrospectiva-sprint-4]]
