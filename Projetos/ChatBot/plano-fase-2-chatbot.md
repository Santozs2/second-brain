---
tipo: planejamento
projeto: ChatBot
fase: 2
status: em-andamento
tags: [chatbot, whatsapp, django, react, react-flow, fluxo, planejamento]
---

> [!info] O que é isso
> Plano da **Fase 2 — construtor de chatbot** (editor visual de fluxo + motor de execução). O usuário optou por antecipar a construção (S1 e S2 já feitos, ver [[#Progresso]]). A **regra de ouro** original era validar a Fase 1 com 1–2 clientes reais antes disto — decisão consciente de seguir em paralelo.

> [!success] Progresso (atualizado 2026-07-20)
> - **S1 — Dados + CRUD** ✅ (commit `1f64e61`). Models `Flow`/`FlowSession` (migration `flows/0001`), `FlowSerializer` + `FlowView(ModelViewSet)` com `activate`/`deactivate`, rota `/api/flows/`. Isolamento multi-tenant testado (invasão PASS).
> - **S2 — Motor básico** ✅ (commit `f343563`). `flows/engine.py`: helpers de leitura do grafo, `_send_bot_text` (mensagens do bot com `sender_membership=None`), `run_session` (loop com trava `MAX_STEPS=50`), `trigger_inbound` (guards: humano/resolvida/sem-fluxo/sessão-existente). Fiado no webhook (`whatsapp/services._process_inbound`) com import preguiçoso pra evitar ciclo. Testado com rollback + mocks: 5 cenários PASS.
> - **S3 — Interação** ✅ (commit `1ba7c64`). Nós `question` (pausa em `waiting_input`, guarda resposta no `context`) e `condition` (ramifica por `sourceHandle` com `eq`/`contains`); `resume_session` retoma a sessão pausada; esquema dos nós formalizado. Testado com rollback + mocks: **16/16 PASS**. Detalhes em [[fase2-s3-question-condition]].
> - **S4 — Handoff** ✅ (commit `230f6aa`). Nó `handoff`: envia texto opcional, seta sessão `HANDED_OFF` (terminal) e conversa `PENDING` (volta pra fila do inbox). Bot fica calado depois de transferir (sessão `HANDED_OFF` ≠ `waiting_input` → `trigger_inbound` sai cedo). Testado com rollback + mocks: **12/12 PASS**. Detalhes em [[fase2-s4-handoff]].
> - **S5 — Editor React Flow** 🔜 PRÓXIMO: paleta de nós, config por nó, salvar/publicar (usa o CRUD `/api/flows/` da S1).

# Fase 2 — Construtor de Chatbot

## 1. Visão
Permitir que cada organização (tenant) monte um **fluxo de atendimento automático** por WhatsApp num editor visual (arrastar nós, ligar setas), e que o sistema **execute esse fluxo** quando um contato manda mensagem — respondendo sozinho, capturando dados e passando pro atendente humano quando necessário.

Duas metades:
- **Editor** (frontend): tela visual estilo fluxograma (React Flow).
- **Motor** (backend): interpreta o fluxo salvo e reage às mensagens de entrada.

## 2. Como conversa com a Fase 1
A Fase 2 se pluga no que já existe, não substitui:
- Gatilho = o mesmo **webhook inbound** (`whatsapp/services.handle_webhook_payload`) que hoje cria `Message`/`Conversation`. Depois de salvar a mensagem, o motor decide se o bot responde.
- **Handoff pro humano** = reusa `assigned_to` e `status` da `Conversation` (passo 2). Quando o fluxo transfere pra humano, o bot pausa e a conversa cai na fila do inbox.
- **Janela de 24h**: o bot só age reagindo a uma mensagem de entrada, então está sempre DENTRO da janela — sem esbarrar no bloqueio de template. Disparo proativo (fora da janela) fica FORA do MVP.
- **Envio**: reusa `whatsapp/client.send_text_message`. Mensagens do bot ficam com `sender_membership=null` + um marcador de origem "bot".

## 3. Modelo de dados (proposta)

> [!tip] Decisão-chave: guardar o grafo como JSON, não em tabelas normalizadas
> React Flow já trabalha com `nodes[]` e `edges[]` em JSON. Guardar isso como um blob (`definition`) é muito mais simples que modelar `FlowNode`/`FlowEdge` em tabelas separadas, e casa 1:1 com o export do editor. O estado de execução (por conversa) fica numa tabela à parte.

- **`Flow`** (`TenantModel`): `name`, `channel` (FK WhatsAppChannel), `definition` (JSONField = `{nodes, edges}`), `is_active` (bool — só 1 ativo por canal), `version` (int, pra histórico futuro).
- **`FlowSession`** (`TenantModel`): estado runtime por conversa. `conversation` (OneToOne/FK), `flow` (FK), `current_node_id` (str), `context` (JSONField = variáveis capturadas, ex.: `{"nome": "Ana", "cpf": "..."}`), `status` (running / waiting_input / handed_off / finished).

Por que `FlowSession` separado: um fluxo é um molde reutilizável; cada conversa tem sua própria "posição" e suas próprias variáveis.

## 4. Tipos de nó (MVP)
| Nó | O que faz |
|---|---|
| **start** | ponto de entrada; 1 por fluxo |
| **send_message** | manda um texto e segue pra próxima seta |
| **question** | manda uma pergunta, **espera** a resposta e salva numa variável do `context` |
| **condition** | ramifica conforme a variável / palavra-chave (ex.: contém "sim") |
| **handoff** | transfere pro humano (seta `status`/`assigned_to`), **para o bot** |
| **end** | encerra o fluxo |

Fora do MVP (fase posterior): mídia, delay/espera temporizada, integração HTTP externa, menu com botões/listas interativas do WhatsApp.

## 5. Motor de execução (backend)
Fluxo de uma mensagem de entrada:
1. Webhook salva a `Message` inbound (como hoje).
2. Motor checa: existe `Flow` ativo no canal? A conversa **não** está com humano (`assigned_to`/`status`)? Se sim, segue.
3. Carrega/cria a `FlowSession` da conversa (posição atual ou nó `start`).
4. **Se estava `waiting_input`**: pega o texto da mensagem, salva na variável do nó `question`, avança pela seta.
5. Roda os nós em sequência até parar num nó que "bloqueia" (`question` → espera; `handoff` → para; `end` → encerra). `send_message` e `condition` passam direto.
6. Cada `send_message` chama `send_text_message` e cria a `Message` out (marcada como bot).
7. Persiste `current_node_id` + `context` na `FlowSession`.

> [!warning] Riscos que o motor PRECISA tratar
> - **Loop infinito** no grafo (setas em ciclo sem `question`/`end`) → guard de nº máximo de passos por execução.
> - **Corrida** de mensagens rápidas em sequência (2 webhooks quase juntos mexendo na mesma `FlowSession`) → lock/idempotência (ex.: `select_for_update` ou processar por ordem).
> - **Fluxo mal montado** (nó sem saída, variável inexistente na condição) → falhar de forma segura e, no limite, cair pro handoff humano.

## 6. Frontend — editor (React Flow)
- Página nova, separada do inbox (ex.: rota `/flows`).
- **Canvas** React Flow: arrastar nós da paleta, ligar com setas.
- **Paleta** lateral com os tipos de nó do MVP.
- **Painel de config** ao clicar num nó (texto da mensagem, nome da variável, regra da condição).
- **Salvar** = POST do JSON `{nodes, edges}` pro `Flow.definition`.
- **Ativar/publicar** = toggle `is_active` (garante 1 ativo por canal).
- Validação no salvar: tem `start`? Todo nó (menos `end`/`handoff`) tem saída?

## 7. Ordem sugerida de sprints
1. **S1 — Dados + CRUD**: models `Flow`/`FlowSession`, endpoints CRUD do fluxo, toggle de ativação. Sem execução ainda.
2. **S2 — Motor básico**: gatilho no webhook + nós `start`/`send_message`/`end`. Testar com fluxo montado "na mão" (JSON no banco).
3. **S3 — Interação**: nós `question` (espera + captura em `context`) e `condition` (ramificação). Trata `waiting_input`.
4. **S4 — Handoff**: nó `handoff` integrado ao inbox (pausa bot, seta status/assigned, notifica atendente via WebSocket que já existe).
5. **S5 — Editor visual**: React Flow (paleta, config, salvar, publicar). Aqui o produto fica "montável" pelo cliente.

> [!note] Estratégia
> Construir o **motor antes do editor**. Com S1–S4 dá pra rodar um fluxo de verdade montando o JSON na mão — valida a lógica sem depender da UI (que é a parte mais cara). O editor visual (S5) é a cereja, não a fundação.

## 8. Perguntas em aberto (decidir antes de codar)
- Identidade das mensagens do bot: só `sender_membership=null`, ou adicionar um campo/flag `is_bot` na `Message`?
- 1 fluxo por canal (simples) ou múltiplos com gatilhos/roteamento (complexo)? → MVP: **1 ativo por canal**.
- O que dispara o bot: TODA conversa nova, ou só quando não há atendente? → MVP: **só se não há humano na conversa**.
- Reabrir conversa resolvida reinicia o fluxo? → provável: nova mensagem após `finished`/`resolved` cria nova `FlowSession` do `start`.

## Links
- [[guia-fase-1-inbox]]
- [[memoria-tecnica-claude]]
