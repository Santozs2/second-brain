---
title: "Retrospectiva — Sprint 2: WhatsApp Cloud API (Webhook + Client)"
aliases: ["Retrospectiva — Sprint 2: WhatsApp Cloud API (Webhook + Client)"]
tags: [django, sprint-2, retrospectiva, whatsapp, cloud-api, webhook, chatbot]
status: concluido
projeto: ChatBot
criado: 2026-07-07
---

> [!info] Projeto: [[ChatBot|💬 ChatBot]] · Tecnologias: [[Django|Django]] · [[WhatsApp Cloud API|WhatsApp Cloud API]] · [[Webhooks|Webhooks]]

# 📋 Retrospectiva — Sprint 2: WhatsApp Cloud API

> [!success] Status: CONCLUÍDO (validado com payload simulado)
> Webhook GET/POST no ar, client de envio escrito e o fluxo de inbound (mensagem da Meta → Contact + Conversation + Message) **testado e aprovado com payload simulado**. Falta só o teste real, que depende de conta na Meta — adiado de propósito.

## 🎯 Objetivo da Sprint
Conectar o backend à WhatsApp Cloud API: receber mensagens dos clientes (webhook) e ter o client pronto pra enviar respostas — a ponte entre o WhatsApp e o inbox.

---

## ✅ O que foi entregue

- `whatsapp/client.py` — `send_text_message(channel, to, text)` faz POST na Graph API e retorna o `wa_message_id` (código pronto, envio real pendente da Meta)
- `whatsapp/webhook.py` — `WhatsAppWebhookView` (`AllowAny`, sem autenticação):
  - **GET**: valida `hub.verify_token` e devolve `hub.challenge` (verificação da Meta)
  - **POST**: sempre responde 200 e loga exceções (pra Meta não reenviar em loop)
- `whatsapp/services.py` — `handle_webhook_payload`: resolve o tenant pelo `metadata.phone_number_id` → `WhatsAppChannel`
  - Inbound: cria/atualiza `Contact` → `Conversation` → `Message` com idempotência por `wa_message_id`
  - Statuses: atualizam a `Message` sem regredir (rank `queued < sent < delivered < read/failed`)
- Rota `api/whatsapp/webhook/` registrada
- Config via env: `WHATSAPP_API_VERSION` (default `v21.0`) e `WHATSAPP_WEBHOOK_VERIFY_TOKEN` (obrigatória)
- **Fluxo de inbound testado e validado** com payload simulado da Meta (via curl)

---

## 🧭 Decisões técnicas tomadas

| Decisão | Escolha | Motivo |
|---------|---------|--------|
| Resolver o tenant no webhook | Via `metadata.phone_number_id` → `WhatsAppChannel` | O webhook não tem header `X-Organization`; o `phone_number_id` é o que identifica a empresa |
| Auth do endpoint | `AllowAny` + `authentication_classes=[]` | A Meta não manda JWT; a "autenticação" é o verify token no GET |
| Resposta do POST | Sempre 200, mesmo com erro (logando) | Se devolver erro, a Meta reenvia o evento em loop |
| Idempotência | Chave `wa_message_id` | A Meta pode reentregar o mesmo evento; não pode duplicar mensagem |
| Status sem regressão | Rank `queued<sent<delivered<read` | Eventos podem chegar fora de ordem; um `sent` atrasado não pode sobrescrever um `read` |
| Criptografia do `access_token` | **Adiada** até ter token real | Sem conta Meta não há token pra proteger; evita gerir chave à toa agora |
| Teste real (ngrok + número) | **Adiado** até ter conta Meta | Dá pra validar ~80% da sprint com payload simulado |

---

## 🔧 Dificuldades encontradas

### 1. curl no CMD do Windows não aceita aspas simples
- **Sintoma:** o POST do webhook voltava 200, mas **nada era gravado** no banco; a query de verificação dava `AttributeError: 'NoneType' object has no attribute 'direction'`.
- **Causa:** no **CMD** do Windows, aspas simples `'...'` **não** delimitam string (só aspas duplas). O JSON do `-d '...'` foi enviado quebrado, o webhook recebeu payload inválido, respondeu 200 (comportamento by design) e não gravou nada.
- **Solução:** rodar os testes no **Git Bash** (aspas simples funcionam) — ou, no CMD, usar aspas duplas por fora e escapar as internas com `\"`.
- **Confirmação:** o mesmo curl rodado no bash gravou certo (`Mensagem: in | 'Ola, teste' | contato: Joao`), provando que o código estava correto e o problema era só o shell.

### 2. `.venv/Scripts/python.exe` não reconhecido no CMD
- **Sintoma:** `'.venv' não é reconhecido como um comando interno ou externo`.
- **Causa:** barra `/` no início do caminho não funciona no CMD (ele espera `\`).
- **Solução:** como o venv já aparece ativado no prompt (`(.venv)`), basta usar `python` direto. Ou trocar `/` por `\`. (Reforço: Git Bash evita isso.)

---

## 🔁 Padrão recorrente identificado

> [!warning] Lição principal da Sprint
> Os dois tropeços foram de **ambiente/shell**, não de lógica — o código Django estava certo desde o começo. O sintoma enganava: um HTTP 200 dava a impressão de sucesso, mas o payload nunca chegou íntegro.
>
> **Aprendizado:** ao testar webhooks no Windows, use **Git Bash**, não CMD. E ao ver "200 mas nada gravou", desconfiar primeiro do **payload que saiu do cliente** (curl/shell) antes de suspeitar do servidor.

---

## 🧪 Teste que fechou a Sprint (payload simulado)

Sem conta na Meta, o inbound foi validado simulando o JSON que a Meta envia:

```bash
# 1. GET (verificação) — devolve o challenge
curl "http://127.0.0.1:8000/api/whatsapp/webhook/?hub.mode=subscribe&hub.verify_token=<TOKEN>&hub.challenge=DESAFIO123"
# → DESAFIO123  ✅

# 2. POST (mensagem recebida)
curl -X POST "http://127.0.0.1:8000/api/whatsapp/webhook/" -H "Content-Type: application/json" \
  -d '{"object":"whatsapp_business_account","entry":[{"changes":[{"value":{"metadata":{"phone_number_id":"TEST_PNID_123"},"contacts":[{"profile":{"name":"Joao"},"wa_id":"5511888887777"}],"messages":[{"from":"5511888887777","id":"wamid.T1","type":"text","text":{"body":"Ola, teste"}}]}}]}]}'
# → 200

# 3. Conferência no banco
# → Mensagem: in | 'Ola, teste' | contato: Joao  ✅
```

**Resultado:** inbound é recebido, resolve o canal pelo `phone_number_id`, cria contato/conversa/mensagem e é idempotente (reenvio não duplica). Fluxo aprovado.

---

## 📌 Estado final

- [x] Client `send_text_message` escrito (envio real pendente da Meta)
- [x] Webhook GET (verificação `hub.challenge`)
- [x] Webhook POST (inbound + statuses)
- [x] Tenant resolvido pelo `phone_number_id`
- [x] Idempotência por `wa_message_id`
- [x] Status sem regressão
- [x] Config por env (`WHATSAPP_API_VERSION`, `WHATSAPP_WEBHOOK_VERIFY_TOKEN`)
- [x] Inbound testado com payload simulado
- [ ] Criptografar `access_token` (adiado — precisa token real)
- [ ] Teste real (ngrok + número de teste da Meta)

---

## 🚀 Próximos passos

**Quando tiver conta na Meta (fecha o que ficou pendente da Sprint 2):**
1. Criptografar `access_token` (`EncryptedTextField` — lib já instalada; falta `FIELD_ENCRYPTION_KEY` + `INSTALLED_APPS` + migration)
2. `ngrok` pra expor o localhost em HTTPS
3. Configurar o webhook no painel da Meta com o verify token
4. Teste real: mandar mensagem do celular → ver chegar no banco → responder pelo client → ver o status evoluir (`sent → delivered → read`)

**Sprint 3 — Tempo real:**
- WebSocket (Django Channels) pra empurrar mensagens novas pro inbox sem refresh

---

## 📚 Referências

- [[Retrospectiva — Sprint 1: Fundação e Multi-Tenancy]]
- [[Guia de Implementação — Fase 1: Inbox Multi-Atendente]]
- [[Multi-Tenancy no Django — Implementação Shared DB, Shared Schema]]
- [[Roadmap — Plataforma de Atendimento WhatsApp]]
