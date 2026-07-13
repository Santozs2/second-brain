---
title: "Guia de Implementação — Fase 1: Inbox Multi-Atendente"
aliases: ["Guia de Implementação — Fase 1: Inbox Multi-Atendente", "Estrutura Django — Fase 1"]
tags: [django, whatsapp, inbox, fase-1, guia, chatbot]
status: em-desenvolvimento
projeto: ChatBot
criado: 2026-07-06
---

> [!info] Projeto: [[ChatBot|💬 ChatBot]] · Tecnologias: [[Django|Django]] · [[Django REST Framework|DRF]] · [[cs-websocket|WebSocket]] · [[JWT|JWT]] · [[React|React]]

# 🏗️ Fase 1 — Inbox Multi-Atendente

> [!abstract] Objetivo da fase
> Ter um inbox compartilhado funcionando: receber e enviar mensagens no WhatsApp via Cloud API, com vários atendentes atuando no mesmo número, conversas atribuíveis e atualização em tempo real. **Ao final desta fase você já tem um produto vendável.**

> [!tip] Como usar este guia
> Segue os sprints na ordem. Cada um depende do anterior. Não pule pro tempo real (Sprint 3) antes de receber/enviar mensagem funcionando (Sprint 2). Marque os checkboxes conforme avança.

---

## 🗺️ Visão geral dos sprints

| Sprint | Foco | Duração estimada |
|--------|------|------------------|
| 1 | Fundação: setup + auth + multi-tenancy | 4–6 dias |
| 2 | Integração Cloud API: enviar/receber + webhook | 5–7 dias |
| 3 | Inbox em tempo real (WebSocket) | 4–5 dias |
| 4 | Frontend do inbox | 5–7 dias |

**Regra:** ao fim de cada sprint, tudo tem que rodar e ser testável. Nada de "termino tudo e testo no final".

---

## 🧱 Sprint 1 — Fundação

### Objetivo
Projeto Django de pé, usuário logando, multi-tenancy isolando dados por organização.

### Tarefas
- [x] Criar projeto Django + apps (`accounts`, `organizations`, `whatsapp`, `contacts`, `inbox`, `common`) ✅ 2026-07-06
- [x] `common/models.py`: `TimeStampedModel` (UUID + timestamps) e `TenantModel` (FK organization) ✅ 2026-07-06
- [x] `accounts`: User customizado (login por email) + UserManager ✅ 2026-07-06
- [x] `organizations`: models `Organization` e `Membership` (papéis owner/admin/agent) ✅ 2026-07-06
- [x] Multi-tenancy: `TenantMiddleware` (injeta `request.tenant`) + `TenantManager` (`for_tenant`) ✅ 2026-07-06
- [x] Auth JWT com `djangorestframework-simplejwt`: endpoints de registro, login, refresh ✅ 2026-07-06
- [x] Configurar `drf-spectacular` (Swagger em `/api/schema/swagger-ui/`) ✅ 2026-07-06
- [x] Rodar `makemigrations` + `migrate` + criar superuser ✅ 2026-07-06

### Pontos de atenção
> [!warning] Ordem importa
> Defina o `AUTH_USER_MODEL = "accounts.User"` **antes** da primeira migration. Trocar o User depois de migrar é doloroso.

- O RBAC (owner/admin/agent) vive no `Membership`, não no User — um mesmo usuário pode ter papéis diferentes em orgs diferentes.
- No fluxo de registro, crie a `Organization` **e** a `Membership` (role=owner) numa transação atômica. O primeiro usuário de uma org é sempre o dono.

### ✅ Definition of Done
- Consigo registrar um usuário, que cria uma organização e vira owner.
- Consigo logar e receber um token JWT.
- Swagger lista os endpoints de auth.
- Dois usuários em orgs diferentes não enxergam dados um do outro (teste manual no shell).

> [!note] Referências
> - [[Multi-Tenancy no Django — Implementação Shared DB, Shared Schema]]
> - [[Stack Final e Estrutura de Repositório — ChatBot WhatsApp]]

---

## 🔌 Sprint 2 — Integração WhatsApp Cloud API

### Objetivo
Receber mensagens que chegam no número e conseguir responder pela API. Esse é o sprint mais técnico — reserve tempo pra depurar o webhook.

### Conceitos que você PRECISA entender antes
- **Webhook de verificação (GET):** a Meta chama sua URL com `hub.mode`, `hub.verify_token` e `hub.challenge`. Você valida o token e **devolve o `hub.challenge` como resposta** (texto puro). Sem isso, a Meta não ativa o webhook.
- **Webhook de eventos (POST):** a Meta manda um JSON com `entry[].changes[].value`. Dentro vem `messages[]` (mensagens que o cliente enviou) **ou** `statuses[]` (atualização de entrega/leitura de mensagens que você enviou). Nunca os dois no mesmo evento.
- **Envio:** `POST https://graph.facebook.com/v{VERSAO}/{phone_number_id}/messages` com header `Authorization: Bearer {access_token}` e corpo JSON.

> [!warning] Versão da API
> Use a versão que aparece no painel do seu app na Meta (ex: `v21.0`, `v22.0`...). Não fixe uma versão velha — confira no dashboard e coloque numa variável de ambiente `WHATSAPP_API_VERSION`.

### Tarefas
- [x] `whatsapp/models.py`: model `WhatsAppChannel` (phone_number_id, waba_id, access_token, verify_token, status) — já existe, revisar ✅ 2026-07-08
- [x] `whatsapp/client.py`: função `send_text_message(channel, to, text)` que faz o POST na Cloud API e retorna o `wa_message_id` ✅ 2026-07-08
- [x] `whatsapp/webhook.py` (ou view): endpoint **GET** que valida `hub.verify_token` e devolve `hub.challenge` ✅ 2026-07-08
- [x] Mesmo endpoint **POST**: recebe os eventos, identifica se é `messages` ou `statuses` ✅ 2026-07-08
- [x] Processar inbound: achar/criar `Contact` (pelo `wa_id`), achar/criar `Conversation`, criar `Message` (direction=in), atualizar `last_inbound_at` ✅ 2026-07-08
- [x] Processar status: achar a `Message` pelo `wa_message_id` e atualizar o `status` (sent→delivered→read / failed) ✅ 2026-07-08
- [x] Expor o `access_token` fora do texto puro (criptografia em repouso — ver nota) ✅ 2026-07-08
- [x] Testar com o **número de teste** que a Meta fornece no painel

### Fluxo de uma mensagem recebida (mental model)
```
Cliente manda msg no WhatsApp
   ↓
Meta faz POST no seu /webhook/
   ↓
Você lê entry[].changes[].value.messages[0]
   ↓
Acha/cria Contact (por wa_id) na org do canal
   ↓
Acha/cria Conversation aberta desse contato
   ↓
Cria Message (direction=in, type=text, wa_message_id)
   ↓
Atualiza conversation.last_inbound_at = agora
   ↓
(Sprint 3) dispara evento WebSocket pro inbox
```

> [!warning] Segurança do token
> `access_token` é credencial sensível. Não deixe em texto puro em produção — use `django-encrypted-model-fields` ou um secrets manager. Deixe isso resolvido ANTES de conectar um número real de cliente.

> [!warning] Webhook precisa de HTTPS público
> Em dev, use `ngrok` (ou similar) pra expor seu `localhost:8000` num HTTPS que a Meta consiga chamar. Em produção, domínio com TLS.

### ✅ Definition of Done
- Mando mensagem do meu celular pro número de teste → aparece uma `Message` inbound no banco.
- Chamo `send_text_message` → a mensagem chega no meu WhatsApp.
- O status da mensagem enviada evolui (sent → delivered → read) conforme os webhooks chegam.

---

## ⚡ Sprint 3 — Inbox em tempo real

### Objetivo
Vários atendentes vendo as conversas atualizarem ao vivo, atribuindo conversas e respondendo — sem dar F5.

### Tarefas
- [x] Revisar models `Conversation` e `Message` (status, assigned_to, janela de 24h) ✅ 2026-07-13
- [x] REST endpoints: listar conversas (filtrado por tenant), listar mensagens de uma conversa, enviar mensagem ✅ 2026-07-13
- [x] Endpoint de **atribuir conversa** a um atendente (`assigned_to`) ✅ 2026-07-13
- [x] Configurar Django Channels no `asgi.py` + `channels_redis` como camada ✅ 2026-07-13
- [x] Criar um **consumer** WebSocket que autentica o usuário e o coloca num grupo por organização (`org_{id}`) ✅ 2026-07-13
- [x] No recebimento de mensagem (Sprint 2) e no envio, fazer **broadcast** pro grupo da org ✅ 2026-07-13
- [x] Implementar a lógica da **janela de 24h**: se `is_within_service_window` é True, permite texto livre; se False, exige template ✅ 2026-07-13

### A regra da janela de 24h
```
Ao tentar enviar uma mensagem numa conversa:
   ↓
conversation.is_within_service_window ?
   ├─ SIM (última msg do cliente < 24h) → envia texto livre normalmente
   └─ NÃO (passou de 24h) → BLOQUEIA texto livre,
          só permite enviar via template aprovado pela Meta
```

> [!note] Templates ficam pra depois
> Na Fase 1 você só precisa **detectar** a janela e bloquear/avisar. A criação e envio de templates aprovados é detalhado na Fase 4 (campanhas). Aqui basta a UI dizer "fora da janela de 24h, use um template".

### Padrão de broadcast (mental model)
```
Novo Message criado (inbound ou outbound)
   ↓
Serializa a mensagem
   ↓
channel_layer.group_send("org_{id}", {novo_evento})
   ↓
Todos os atendentes conectados daquela org recebem no WebSocket
   ↓
Frontend insere a mensagem na conversa aberta em tempo real
```

> [!warning] Isolamento no WebSocket também
> O grupo do Channels é por organização (`org_{id}`). Nunca use um grupo global — senão um atendente de uma empresa recebe mensagem de outra. O `request.tenant` do WebSocket vem do usuário autenticado no handshake.

### ✅ Definition of Done
- Abro o inbox em duas abas (dois atendentes da mesma org) → mensagem nova aparece nas duas ao vivo.
- Atribuo uma conversa a um atendente e isso reflete pros outros.
- Tento responder uma conversa parada há mais de 24h → o sistema me avisa que preciso de template.

---

## 🎨 Sprint 4 — Frontend do inbox

### Objetivo
A interface que o atendente usa: lista de conversas à esquerda, conversa aberta no centro, campo de envio embaixo — atualizando em tempo real.

### Tarefas
- [x] Setup React + Vite + TypeScript + Tailwind + Shadcn/ui ✅ 2026-07-13
- [x] Auth no front: tela de login, guardar token, `ProtectedRoute`, interceptor do axios pra mandar o Bearer ✅ 2026-07-13
- [x] Layout base (sidebar + header + área principal) ✅ 2026-07-13
- [x] **Lista de conversas** (ordenada por `last_message_at`, com status e atendente) ✅ 2026-07-13
- [x] **Detalhe da conversa**: histórico de mensagens (inbound à esquerda, outbound à direita) ✅ 2026-07-13
- [x] **Campo de envio** (texto + futuramente mídia) ✅ 2026-07-13
- [x] Cliente WebSocket: conecta, escuta o grupo da org, insere mensagens ao vivo ✅ 2026-07-13
- [x] Ação de **atribuir conversa** a um atendente ✅ 2026-07-13
- [x] Indicador visual da **janela de 24h** (badge "fora da janela" quando aplicável) ✅ 2026-07-13

### Ordem sugerida de construção do front
1. Login + rota protegida (sem isso nada funciona)
2. Lista de conversas (só leitura, via REST)
3. Abrir conversa + ver mensagens (via REST)
4. Enviar mensagem (via REST) → confirma que chega no WhatsApp
5. **Só então** plugar o WebSocket pro tempo real
6. Atribuição + badge da janela de 24h

> [!tip] Não comece pelo tempo real
> É tentador, mas frustra. Faça tudo via REST primeiro (request/response, fácil de debugar). Quando a tela já funciona no F5, aí você adiciona o WebSocket por cima. Muito mais fácil de achar bug.

### ✅ Definition of Done
- Logo, vejo a lista de conversas da minha org.
- Abro uma conversa, leio o histórico, respondo, e a resposta chega no WhatsApp real.
- Mensagem nova do cliente aparece na tela sem eu recarregar.
- Consigo atribuir uma conversa a um atendente.

---

## 🏁 Critério de conclusão da Fase 1

> [!success] Fase 1 concluída quando...
> Um atendente loga, vê as conversas do número da empresa, responde clientes reais pelo WhatsApp, mensagens novas aparecem ao vivo, e conversas podem ser distribuídas entre atendentes. **Isso já é vendável — hora de validar com 1–2 clientes reais antes de partir pra Fase 2 (chatbot).**

### Checklist final da fase
- [x] Multi-tenancy isolando dados por organização (testado com 2 orgs)
- [x] Auth JWT funcionando (registro, login, refresh)
- [x] RBAC (owner/admin/agent) aplicado
- [~] Webhook da Meta recebendo mensagens e status *(implementado e testado com payload simulado; falta validar com a Meta real)*
- [~] Envio de mensagem via Cloud API *(`send_text_message` pronto; falta testar com número real)*
- [x] Inbox em tempo real (WebSocket por org)
- [x] Atribuição de conversa a atendente
- [x] Detecção da janela de 24h
- [~] Frontend do inbox usável de ponta a ponta *(leitura + tempo real ok; envio pelo atendente depende da Meta)*

---

## ⚠️ Armadilhas comuns (leia antes de começar)

- **Trocar o User model depois de migrar** → refaça o banco do zero se precisar mudar. Defina o custom User no Sprint 1.
- **Webhook não ativa** → quase sempre é o `verify_token` que não bate, ou a URL não está em HTTPS público. Cheque o `ngrok` e o token.
- **"Mensagem some"** → webhook de eventos não configurado/assinado no painel da Meta. Assine os campos `messages`.
- **Passar `organization_id` pelo client** → nunca. Sempre derive do `request.tenant`. É o buraco de segurança mais fácil de deixar aberto.
- **Grupo WebSocket global** → sempre por org. Senão vaza conversa entre empresas.
- **Deixar token da Meta em texto puro** → resolva a criptografia antes de plugar cliente real.

---

## 📚 Referências

- [[Roadmap — Plataforma de Atendimento WhatsApp]]
- [[Multi-Tenancy no Django — Implementação Shared DB, Shared Schema]]
- [[Stack Final e Estrutura de Repositório — ChatBot WhatsApp]]
