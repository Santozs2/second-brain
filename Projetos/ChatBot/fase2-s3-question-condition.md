---
tipo: nota-tecnica
projeto: ChatBot
fase: 2
sprint: S3
status: concluido
commit: 1ba7c64
tags: [chatbot, whatsapp, django, fluxo, motor, question, condition, contexto]
---

> [!success] S3 concluída — 2026-07-20 (commit `1ba7c64`)
> Nós **`question`** e **`condition`** no motor de fluxo, com contexto de variáveis e retomada de sessão pausada. Testado com rollback + mocks: **16/16 checks PASS**.

# Fase 2 · S3 — Interação (`question` + `condition`)

## O que a sprint entrega
O motor deixou de ser só linear (`start → send_message → end`) e passou a **conversar**: faz uma pergunta, **espera** a resposta do cliente, guarda numa variável e **ramifica** o fluxo conforme o que veio.

## Esquema dos nós (contrato motor ↔ editor)
Tudo vive em `Flow.definition = {"nodes": [...], "edges": [...]}` (formato React Flow). Todo nó tem `id`, `type`, `data`.

| `type` | Campos em `data` | O que o motor faz | Pausa? | Saídas |
|---|---|---|---|---|
| `start` | — | ponto de entrada (1 por fluxo) | não | 1 |
| `send_message` | `text` | envia o texto pelo bot | não | 1 |
| `question` | `text`, `variable` | envia `text`, guarda a próxima resposta em `context[variable]` | **sim** (`waiting_input`) | 1 |
| `condition` | `variable`, `op` (`eq`\|`contains`), `value` | compara `context[variable]` com `value`, escolhe o ramo | não | N (por rótulo) |
| `end` | — | encerra (`finished`) | não | 0 |

**Edges**: `source`, `target` e — só no `condition` — `sourceHandle` (`"true"` / `"false"` / `"default"`).

### Regras do `condition`
- `op: "eq"` → verdadeiro se `context[variable] == value`.
- `op: "contains"` → verdadeiro se `value` está dentro de `context[variable]` (**case-insensitive** — cobre "quero falar com ATENDENTE").
- Resultado escolhe o edge com `sourceHandle` correspondente; se faltar, cai no `"default"`; sem nenhum → encerra.
- Variável ausente conta como `""` (sem `KeyError`).

## Implementação (`flows/engine.py`)
- **`run_session`** — ordem do loop: `end → question → condition → send_message → avanço linear`.
  - `question`: envia o texto, salva `current_node_id` no **próprio nó da pergunta**, seta `WAITING_INPUT` e `return` (única forma de pausar).
  - `condition`: `_eval_condition` → `_branch_node_id` → `continue` (não manda mensagem, reprocessa o novo nó na mesma execução).
- **`resume_session(session, answer_text)`** (nova) — grava `context[variable]=resposta`, avança pelo `_next_node_id`, volta pra `RUNNING`, chama `run_session`. Salva incluindo `"context"` no `update_fields` (mutar o `JSONField` em memória não persiste sozinho).
- **`_eval_condition(data, context)`** — devolve a string `"true"`/`"false"` (casa direto com `sourceHandle`).
- **`_branch_node_id(definition, node_id, handle)`** — escolhe o edge pelo rótulo, com fallback `"default"`.
- **`trigger_inbound(conversation, text)`** — passou a receber o **texto**. Se já há sessão em `waiting_input` → `resume_session`; senão cai nos guards de sempre (humano/resolvida/fluxo ativo/start) e cria sessão nova.
- **`whatsapp/services.py`** — `_process_inbound` passa `msg.text` ao motor.

## Fluxo de exemplo (menu de triagem)
`start → question("1 suporte ou 2 vendas?", var=opcao)` → **pausa**. Cliente responde `"1"` → `resume` grava `opcao="1"` → `condition eq "1"` → ramo `true` → `send_message` Suporte → `end`.

## Teste (rollback, nada persistiu no Supabase)
`transaction.set_rollback(True)` + mocks em `send_text_message` e `broadcast_new_message` (sem Meta/Redis). **16/16 PASS**:
- Pausa na `question` (`waiting_input`, para no nó certo, 1 msg, contexto vazio).
- Ramo `true` (resposta "1" → Suporte, `finished`, msgs `SENT` com `wa_message_id`).
- Ramo `false` (resposta "2" → Vendas).
- `contains` case-insensitive ("...ATENDENTE..." → ramo true).
- Guard humano (conversa atribuída não cria sessão).
- Idempotência (sessão `finished` ignora nova mensagem, não reabre).

## Bugs pegos na revisão (padrão que se repete)
> [!warning] Recorrentes
> - **Bloco duplicado no loop**: a versão antiga (`send_message` + avanço) sobrou antes do bloco novo → **duplo avanço** que pulava a `question` (fluxo ia direto pro `finished`). Foi o bug mais escorregadio.
> - `msg.text` no arquivo errado (o `engine` usa o parâmetro `text`; quem tem `msg` é o `services`).
> - Typos: `update_at`→`updated_at`, `FieldsNotExist=`→`update_fields=`, `return None` mal indentado dentro do `for`.
> - Lógica de avanço presa dentro do `if variable` no `resume_session` (travava a sessão se o nó não tivesse `variable`).

## Próximo
- **S4 — Handoff**: nó `handoff` que seta `handed_off` + devolve a conversa pro atendente (reusa `assigned_to`/`status` do inbox + WebSocket). O `condition contains "atendente"` já prepara o gatilho natural.
- **S5 — Editor React Flow**: paleta, config por nó, salvar/publicar.

## Links
- [[plano-fase-2-chatbot]]
- [[guia-fase-1-inbox]]
- [[memoria-tecnica-claude]]
- [[endurecimento-producao]]
