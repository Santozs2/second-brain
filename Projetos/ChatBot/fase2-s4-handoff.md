---
tipo: nota-tecnica
projeto: ChatBot
fase: 2
sprint: S4
status: concluido
commit: 230f6aa
tags: [chatbot, whatsapp, django, fluxo, motor, handoff, atendente]
---

> [!success] S4 concluída — 2026-07-20 (commit `230f6aa`)
> Nó **`handoff`** no motor: transfere a conversa pro atendente humano. Sessão vira `HANDED_OFF` (terminal), conversa vira `PENDING` (fila do inbox), bot fica calado. Testado com rollback + mocks: **12/12 PASS**.

# Fase 2 · S4 — Handoff (transferência pro humano)

## O que a sprint entrega
O fluxo deixou de ser só automático: agora sabe **desistir e chamar gente**. Ao chegar num nó `handoff`, o bot manda um recado final ("vou te transferir"), **encerra a própria atuação** e devolve a conversa pra fila do inbox — de onde um atendente assume.

## Esquema do nó (contrato motor ↔ editor)
| `type` | Campos em `data` | O que o motor faz | Pausa? | Saídas |
|---|---|---|---|---|
| `handoff` | `text` (opcional) | envia `text`, seta sessão `HANDED_OFF` + conversa `PENDING`, encerra a atuação do bot | **sim** (terminal) | 0 |

Diferença pro `question`: `question` pausa esperando resposta (`waiting_input`, retomável); `handoff` pausa **de vez** (`handed_off`, terminal — não retoma).

## Duas mudanças de estado (o pulo do gato)
- **`FlowSession.status = HANDED_OFF`** → estado da *sessão do bot*. É terminal: `trigger_inbound` só retoma sessão em `waiting_input`, então numa `HANDED_OFF` ele **sai cedo** → é isso que mantém o bot calado depois de transferir.
- **`Conversation.status = PENDING`** → estado da *conversa no inbox*. `Conversation` **não tem** `HANDED_OFF` (só `open`/`pending`/`resolved`); `PENDING` ("Aguardando") = caiu na fila esperando atendente. Reusa o fluxo do inbox da Fase 1 sem inventar status novo.

> [!warning] Bug pego na revisão (o de sempre: colar sem apagar)
> Primeira versão usava `conversation.Status.HANDED_OFF` (não existe → `AttributeError`), esquecia o `conversation.save()` (só mutava em memória) e chamava `session.save(update_fields=[...])` **sem antes setar** `current_node_id`/`status` (salvava campos velhos). Correto: setar conversa→`PENDING` + `save`, sessão→`HANDED_OFF` + `current_node_id=node_id` + `save`, então `return`.

## Implementação (`flows/engine.py`)
Um `if` a mais no loop do `run_session`, posição importa — **depois** do `condition`, **antes** do `send_message`:

```python
if node_type == "handoff":
    text = node.get("data", {}).get("text", "")
    if text:
        _send_bot_text(conversation, text)
    conversation.status = conversation.Status.PENDING
    conversation.save(update_fields=["status", "updated_at"])
    session.current_node_id = node_id
    session.status = FlowSession.Status.HANDED_OFF
    session.save(update_fields=["current_node_id", "status", "updated_at"])
    return
```

Ordem final do loop: `end → question → condition → handoff → send_message → avanço linear`.

## Fluxo de exemplo (triagem com escape)
`start → question("Como posso ajudar?", var=intencao)` → **pausa**. Cliente: `"quero falar com um ATENDENTE"` → `resume` grava `intencao` → `condition contains "atendente"` → ramo `true` → `handoff("Vou te transferir")` → sessão `HANDED_OFF`, conversa `PENDING`. Ramo `false` segue linear (`send_message → end`), sem handoff.

## Teste (rollback, nada persistiu no Supabase)
`transaction.set_rollback(True)` + mocks em `send_text_message` e `broadcast_new_message`. **12/12 PASS**:
- Handoff: sessão `HANDED_OFF`, para no nó `handoff`, conversa `PENDING`, texto enviado, msgs `SENT` marcadas como bot (`sender_membership=None`).
- Silêncio pós-handoff: nova mensagem numa sessão `HANDED_OFF` não reabre nem responde (0 msgs novas).
- Ramo `false`: termina `FINISHED`, conversa continua `OPEN` (não vira `PENDING`).

## Próximo
- **S5 — Editor React Flow**: paleta de nós, config por nó (form por `type`), salvar/publicar via o CRUD `/api/flows/` da S1. Fecha a Fase 2 (backend já executa os 6 tipos de nó).

## Links
- [[plano-fase-2-chatbot]]
- [[fase2-s3-question-condition]]
- [[guia-fase-1-inbox]]
- [[endurecimento-producao]]
