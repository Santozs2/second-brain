---
title: "Retrospectiva — Sprint 3: Inbox em Tempo Real (WebSocket)"
aliases: ["Retrospectiva — Sprint 3: Inbox em Tempo Real (WebSocket)"]
tags: [django, sprint-3, retrospectiva, websocket, channels, tempo-real, chatbot]
status: concluido
projeto: ChatBot
criado: 2026-07-08
---

> [!info] Projeto: [[ChatBot|💬 ChatBot]] · Tecnologias: [[Django|Django]] · [[Django Channels|Django Channels]] · [[cs-websocket|WebSocket]] · [[JWT|JWT]]

# 📋 Retrospectiva — Sprint 3: Inbox em Tempo Real (WebSocket)

> [!success] Status: CONCLUÍDO (validado ao vivo num servidor real, com payload simulado)
> Mensagem recebida no webhook aparece **ao vivo** no cliente WebSocket, sem refresh e sem polling. Handshake autenticado por JWT, isolamento por organização e broadcast automático. Validado primeiro em processo e depois **ao vivo no navegador** (`runserver` + Redis), disparando o webhook pelo console. Falta só o teste com a Meta real (adiado) e o consumo pelo frontend (Sprint 4).
>
> [!warning] Correção pós-fechamento: o `InMemoryChannelLayer` **não** entrega o broadcast num servidor real — a migração pra `RedisChannelLayer` (via Docker) foi trazida da produção pro dev ainda nesta sprint. Ver **Dificuldade 4**.

## 🎯 Objetivo da Sprint
Empurrar mensagens novas pro inbox em tempo real: quando a Meta entrega uma mensagem no webhook, os atendentes conectados da empresa recebem na hora, via WebSocket, sem atualizar a página.

---

## ✅ O que foi entregue

- **Django Channels + Daphne** instalados e configurados
  - `INSTALLED_APPS`: `daphne` no topo (substitui o `runserver` pela versão ASGI) + `channels`
  - `ASGI_APPLICATION` apontando pro `chatbot.asgi.application` (WSGI mantido)
  - `CHANNEL_LAYERS`: começou com `InMemoryChannelLayer`, migrou pra `RedisChannelLayer` (`127.0.0.1:6379`) ainda no dev — ver Dificuldade 4
- `chatbot/asgi.py` — `ProtocolTypeRouter` separando `http` (Django) de `websocket` (nossa pilha)
- `common/ws_auth.py` — `JWTAuthMiddleware`: lê o `?token=` da query string, valida com `AccessToken` do SimpleJWT e popula o `scope["user"]` (fallback `AnonymousUser`)
- `inbox/consumers.py` — `InboxConsumer`: autentica no `connect`, resolve a org do usuário e entra no grupo `org_{id}`; handler `inbox_message` empurra o evento pro cliente
- `inbox/routing.py` — rota `ws/inbox/`
- `whatsapp/services.py` — `_broadcast_new_message`: ao criar mensagem inbound, faz `group_send` pro `org_{id}` **após o commit** da transação, com a mensagem já serializada (`MessageSerializer`)
- **Fluxo de tempo real testado e aprovado end-to-end** (handshake + broadcast)

---

## 🧭 Decisões técnicas tomadas

| Decisão | Escolha | Motivo |
|---------|---------|--------|
| Camada de canais (dev e prod) | `RedisChannelLayer` | O `InMemory` não entrega broadcast HTTP→WS num servidor real (webhook roda em thread/loop separado); Redis faz a ponte de rede cross-thread/loop |
| Rodar o Redis | Container **Docker** (`redis:7`) | Redis nativo no Windows é chato; `docker run -d --name chatbot-redis -p 6379:6379 redis:7` sobe em 1 comando e é descartável |
| Versão do `redis-py` | **`redis<6`** (5.3.1) | `redis-py` 8.x é incompatível com `channels_redis` 4.3.0 → WebSocket cai em ~10s com close `1011` (ver Dificuldade 4) |
| Token no WebSocket | Via query string `?token=` | A API de WebSocket do navegador não deixa setar headers customizados no handshake |
| Grupo do Channels | Por org (`org_{id}`), nunca global | Isolamento entre empresas — só quem está no grupo da org recebe a mensagem dela |
| Momento do broadcast | Fora do `transaction.atomic()` (após commit) | Não avisar os atendentes de uma mensagem que pode sofrer rollback |
| Auth no handshake | Middleware JWT custom (não `AuthMiddlewareStack`) | A API é JWT puro, sem sessão; o stack padrão do Channels depende de sessão/cookie |
| `daphne` no topo do `INSTALLED_APPS` | Obrigatório | É ele que troca o `runserver` pela versão ASGI que aceita WebSocket |
| Ponte sync ↔ async | `database_sync_to_async` (ORM no consumer) e `async_to_sync` (group_send no serviço síncrono) | Consumer é async e o ORM é sync; o webhook é sync e o `group_send` é async |

---

## 🔧 Dificuldades encontradas

### 1. `pip` instalou no Python global, não no venv
- **Sintoma:** `pip install channels daphne` dizia "Successfully installed", mas `import channels` continuava dando `ModuleNotFoundError`.
- **Causa:** o `pip` puro resolveu pro Python **global** (`Python314`), não pro venv do projeto.
- **Solução:** instalar sempre via `./.venv/Scripts/python.exe -m pip install <pkg>` (rodando de `backend/`). Aí `channels 4.3.2` e `daphne 4.2.2` foram parar no venv certo.

### 2. Aviso E402 (import fora do topo) no `asgi.py`
- **Sintoma:** o linter reclamou "Module level import not at top of file" nos imports do Channels/consumer.
- **Causa:** **falso positivo** — é o padrão oficial do Channels. Os imports do routing/consumer **têm** que vir depois do `get_asgi_application()`, senão o Django ainda não carregou os apps e dá `AppRegistryNotReady` (o consumer importa o model `Membership`).
- **Solução:** `# noqa: E402` nas 3 linhas. Ordem é de propósito, não bagunça.

### 3. Testar WebSocket sem cliente externo (e sem quebrar o event loop)
- **Sintoma:** não havia lib cliente (`websockets`) no venv; e chamar o webhook (que usa `async_to_sync`) dentro de um teste async esbarraria no "you cannot use async_to_sync in the same thread as an async event loop".
- **Solução:** usei o `WebsocketCommunicator` do próprio Channels (testa o mesmo `application` do `asgi.py`, em processo) e disparei o webhook com `sync_to_async(handle_webhook_payload, thread_sensitive=True)` — assim o `group_send` roda no **mesmo event loop** do communicator, evitando o problema de cross-loop do `InMemoryChannelLayer`.

### 4. O `InMemoryChannelLayer` **não** funciona ao vivo (e o `redis-py` 8 quebra o Redis)
- **Sintoma:** o teste em processo passava, mas no navegador (`runserver` real) a mensagem **não** chegava ao vivo — o webhook retornava `200`, a `Message` era criada, mas o WebSocket ficava mudo.
- **Causa raiz:** o `InMemoryChannelLayer` prende as filas a **um** event loop. A webhook é uma view **sync** (roda numa thread separada); o `async_to_sync(group_send)` executa num loop diferente do consumer (loop principal do Daphne) → a mensagem cai no vazio. O erro é engolido pelo `try/except` da webhook, por isso o `200`. Só funcionava no teste in-process porque eu forçava o mesmo loop.
- **Solução:** trazer o `channels_redis` + Redis (o pub/sub de rede funciona cross-thread/loop/processo) **já pro dev**, não só pra produção. Redis rodando em **container Docker** (`redis:7`), `CHANNEL_LAYERS` apontando pro `RedisChannelLayer`.
- **Bug bônus (close `1011`):** depois de ligar o Redis, o WebSocket conectava e **caía em ~10s** com close `1011` e traceback `redis.exceptions.TimeoutError: Timeout reading from`. Causa: `redis-py` **8.x** é incompatível com `channels_redis` 4.3.0. **Fix:** pinar **`redis<6`** (usei 5.3.1). Depois disso: conecta, segura e entrega ao vivo. ✅
- **Aprendizado:** "InMemory no dev, Redis em prod" é um mito perigoso pro caso HTTP→WebSocket. **Se a origem do broadcast é síncrona (webhook), precisa de Redis mesmo no dev** pra testar de verdade.

---

## 🔁 Padrão recorrente identificado

> [!warning] Lição principal da Sprint
> O coração conceitual do Channels é a **ponte sync ↔ async**. Saber *quando* usar cada direção é o que destrava tudo:
> - **`database_sync_to_async`** → chamar ORM (sync) de dentro de código async (consumer).
> - **`async_to_sync`** → chamar `group_send` (async) de dentro de código sync (webhook/serviço).
>
> E a regra de ouro do multi-tenancy **também vale no WebSocket**: grupo por org (`org_{id}`), nunca global — senão um atendente de uma empresa recebe mensagem de outra. O `org_id` vem do usuário autenticado no handshake, nunca do cliente.

---

## 🧪 Teste que fechou a Sprint (em processo, sem servidor externo)

Usando `channels.testing.WebsocketCommunicator` (exercita o mesmo `application` do `asgi.py`):

**a) Handshake / autenticação**
```
[1] token valido   -> conectou: True   ✅ (esperado True)
[2] sem token      -> conectou: False  ✅ (esperado False)
[3] token invalido -> conectou: False  ✅ (esperado False)
```

**b) Tempo real end-to-end** (conecta cliente → dispara o webhook simulado → recebe ao vivo)
```
1) cliente WebSocket conectou: True
2) webhook disparado (simulando a Meta entregar a mensagem)
3) >>> RECEBIDO AO VIVO:
       texto: "Mensagem em tempo real!"
       direction: in
       conversation_id: 3ec6eff4-...
```

**Fluxo validado:** Meta → webhook → cria `Message` → `group_send("org_{id}")` → consumer (autenticado por JWT) → cliente recebe na hora. Aprovado.

**c) Confirmação ao vivo (servidor real + Redis)** — depois da migração pro `RedisChannelLayer`, o mesmo fluxo foi validado no navegador: `new WebSocket("ws://localhost:8000/ws/inbox/?token=<jwt>")` conecta, dispara o webhook pelo `fetch` no console e a mensagem cai no `onmessage` na hora. Foi isso que confirmou que a limitação era do `InMemory`, não do código.

---

## 📌 Estado final

- [x] Channels + Daphne + ASGI configurados
- [x] `ProtocolTypeRouter` separando http/websocket no `asgi.py`
- [x] Middleware JWT no handshake (`?token=`)
- [x] `InboxConsumer` com grupo por org (`org_{id}`)
- [x] Broadcast automático da mensagem inbound (após commit)
- [x] Handshake testado (válido conecta, sem/ inválido recusa)
- [x] Fluxo de tempo real testado end-to-end com payload simulado (em processo)
- [x] `RedisChannelLayer` + Redis (Docker) rodando **já no dev** — validado ao vivo no navegador
- [x] `redis<6` pinado no `requirements.txt` (evita o close `1011`)
- [ ] Enviar mensagem pelo atendente (`POST` → grava out + envia pela Meta) — esbarra na conta Meta
- [ ] Frontend consumir o WebSocket (Sprint 4)
- [ ] Deploy: servir com Daphne/ASGI de verdade (não o `runserver`)

---

## 🚀 Próximos passos

**Rumo à produção (o essencial da tempo real já está feito):**
1. ~~Trocar `InMemoryChannelLayer` por `channels_redis` + Redis~~ ✅ feito nesta sprint (Redis via Docker, `redis<6`)
2. Servir com Daphne/ASGI de verdade no deploy (não o `runserver`)
3. Subir o Redis no deploy (managed ou container) — no dev ele roda no container `chatbot-redis`

**Sprint 4 — Frontend React:**
- Login (guardar o JWT) e tela do inbox
- Listar conversas e mensagens via REST (endpoints já prontos)
- Conectar no WebSocket (`ws/inbox/?token=<jwt>`) e inserir mensagens ao vivo
- Enviar mensagem pelo atendente (depende do envio real via Meta)

---

## 📚 Referências

- [[Retrospectiva — Sprint 2: WhatsApp Cloud API (Webhook + Client)]]
- [[Retrospectiva — Sprint 1: Fundação e Multi-Tenancy]]
- [[Guia de Implementação — Fase 1: Inbox Multi-Atendente]]
- [[Multi-Tenancy no Django — Implementação Shared DB, Shared Schema]]
- [[Roadmap — Plataforma de Atendimento WhatsApp]]
