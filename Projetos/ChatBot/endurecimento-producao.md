---
tipo: backlog
projeto: ChatBot
data: 2026-07-17
status: mapeado
tags:
  - chatbot
  - producao
  - robustez
  - qualidade
---

# Endurecimento para Produção — Backlog de Qualidade

> [!info] O que é isto
> Feedback crítico do estado da **Fase 1 (Inbox)**: o que separa um MVP que funciona numa demo de um produto confiável que aguenta cliente pagante. A base de **arquitetura e segurança está acima da média** para o estágio; o que falta é a camada de **robustez operacional**. Nada aqui é reescrita — é maturação, mapeada para depois da Fase 2.

Relacionado: [[guia-fase-1-inbox]] · [[seguranca-multi-tenant]] · [[plano-fase-2-chatbot]] · [[roadmap-plataforma-whatsapp]]

## 🔴 Bloqueadores para produção

### 1. Fila assíncrona (Celery/RQ) — o item que muda o jogo
Hoje o **envio** de mensagem é síncrono dentro do request e o **webhook** processa inbound de forma síncrona. Problemas:
- Se a Graph API demora/cai, o atendente trava e **não há retry**.
- A Meta reenvia o webhook se você demora (>~5s) → sob carga dá para processar a **mesma mensagem 2x**. A idempotência por `wa_message_id` cobre inbound duplicado, mas não protege o envio nem operações lentas.
- O motor de fluxo (Fase 2) roda dentro do webhook — quanto mais nós, mais lento o request.

**Solução:** mover envio + processamento de webhook + motor de fluxo para uma **fila com retry** (Celery + Redis, que já está no stack). O webhook vira "recebe → enfileira → 200 na hora".

### 2. Testes automatizados
Todos os testes até aqui foram scripts manuais com rollback, rodados **uma vez e descartados**. Ótimo para validar, péssimo para manter. Sem suíte `pytest` versionada, cada feature nova pode quebrar as anteriores em silêncio — e num SaaS multi-tenant, **uma regressão de isolamento é vazamento de dados de cliente**.

**Mínimo:** isolamento de tenant, webhook (assinatura + idempotência), envio, motor de fluxo.

### 3. RBAC aplicado
Existem papéis `owner/admin/agent` no `Membership`, mas **nenhum endpoint checa**. Um `agent` faz tudo que um `owner` faz (atribuir, resolver, futuramente editar fluxos/canais). Obrigatório para produto real.

## 🟡 Sério, mas não bloqueante

### 4. Concorrência de atendentes
Dois atendentes na mesma conversa (um resolve, o outro responde) — sem lock, sem "está digitando", `assign` não é atômico contra corrida. OK para 1 atendente; problema para equipe.

### 5. Observabilidade
Há `logger.exception` em pontos-chave (bom), mas **nada de métricas, alertas ou rastreio**. Falha de entrega em produção vai ser descoberta pelo cliente reclamando, não pelo sistema avisando. Mensagens `failed` deveriam ser **visíveis no inbox** e idealmente alertar.

### 6. Token permanente da Meta (System User)
Trocar o token na mão no banco é insustentável — o produto **para sozinho** a cada expiração. Trocar por token permanente via System User não é "melhoria futura", é requisito para qualquer cliente real.

### 7. Paginação e performance
`ConversationList` e `messages` carregam **tudo**. Conversa com meses de histórico ou org com milhares de conversas fica lenta e pesada. Precisa de paginação + scroll infinito.

## 🟢 O que já está bom (calibragem)

- **Isolamento multi-tenant** sólido e **auditado** — raro nesse estágio ([[seguranca-multi-tenant]]).
- **Vulnerabilidades reais corrigidas** (HMAC do webhook, `for_tenant(None)` → 500) — postura de segurança madura.
- **Criptografia do `access_token`** at-rest, config por env, secrets fora do git — fundação certa.
- **Arquitetura limpa**: `TenantModel`, camadas services/views separadas, decisões documentadas.

## Ordem de prioridade sugerida (pós-Fase 2)

1. Fila assíncrona (Celery) para webhook + envio + motor, com retry.
2. Suíte de testes mínima: isolamento, webhook, envio, motor.
3. RBAC nos endpoints.
4. Token permanente via System User.
5. Paginação + falhas de envio visíveis no inbox.
6. Observabilidade (métricas + alertas).
7. Concorrência de atendentes (lock/typing).

> [!note] Regra de ouro (do roadmap)
> Validar a Fase 1 com **cliente real** antes de investir pesado. Vários destes itens (token permanente, observabilidade, paginação) só doem de verdade com uso real — priorizar pelo que o primeiro cliente encostar.
