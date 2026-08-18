# Fluxo — Kickoff

Dashboard de finanças pessoais. Projeto solo, usado também pra treinar workflow profissional de GitHub (Issues → branch por feature → PR → CI).

Repo local: `C:\Users\USER\Documents\vs code\fluxo\`
Repo remoto: https://github.com/Santozs2/fluxo (privado)

## Decisões cravadas

| Ponto | Decisão |
|---|---|
| Usuários | Single-user (só eu). Auth JWT simples, sem multi-tenancy. |
| Stack | Django REST Framework + React/Vite/TS/Tailwind + PostgreSQL. |
| MVP | Lançamento manual de receitas/despesas, categorias, contas, dashboard simples. |
| Fora do MVP | Orçamento, metas, import CSV/OFX, cartão de crédito, investimentos — backlog fase 2. |

## Workflow de Git/GitHub (o motivo de treinar esse projeto)

Regra de ouro: **nenhum commit direto na `main`.**

1. **Issue** — toda tarefa nasce como uma Issue no GitHub (ticket descrevendo o que fazer).
2. **Branch por feature** — a partir da `main` atualizada: `git checkout -b feat/nome-da-coisa`.
3. **Pull Request** — ao terminar, push da branch + abrir PR pra `main`. Claude atua como reviewer (comenta, pede ajuste).
4. **CI** — GitHub Actions roda checks (lint/testes/build) em cada PR, antes do merge.
5. **CD** — fase futura: deploy automático após merge na `main` (Render/Railway/Fly.io).

Observação: branch protection nativa da `main` (bloquear push direto) exige GitHub Pro em repo privado — não disponível no plano atual. Vamos seguir a regra por disciplina manual até avaliar deixar o repo público ou upgrade.

Labels usadas nas Issues: `backend`, `frontend`, `sprint-0`, `sprint-1`, `sprint-2` (mais sprints conforme o roadmap avança).

## Arquitetura (visão inicial)

**Backend (apps Django)**
- `accounts` — User por email + JWT
- `finance` — núcleo: Account, Category, Transaction

**Modelo de dados (MVP)**
```
Account      : name, type (checking/savings/wallet/credit_card), initial_balance, currency
Category     : name, kind (income/expense), color
Transaction  : account (FK), category (FK null), date, amount, description, kind
```

## Roadmap por sprints

- **Sprint 0** — Setup: repo privado ✅, CI básico (issue #1)
- **Sprint 1** — Backend: auth JWT + models + API REST (issues #2–#5)
- **Sprint 2** — Frontend: scaffold + login + listagem + form de lançamento (issue #6 em diante)
- **Sprint 3** — Dashboard (saldo total, totais por categoria)
- **Sprint 4** — Polimento + deploy (CD)

Issues abertas: https://github.com/Santozs2/fluxo/issues
