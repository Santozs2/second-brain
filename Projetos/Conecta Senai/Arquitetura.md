---
type: project
tags:
  - project
created: 2026-06-30
updated: 2026-06-30
status: em-andamento
---

# Arquitetura

```mermaid
graph TD
A[Usuário] --> B[Templates HTML]
B --> C[Views Django]
C --> D[Models]
D --> E[(SQLite)]
```

---

## Padrão

Monólito Django (MVC/MVT).

→ [[Conceitos/MVC|MVC]]

---

## Backend

Django é responsável pela lógica de negócio, rotas e painel administrativo.

→ [[Estudos/Django|Django]] · [[Estudos/Python|Python]] · [[Conceitos/Models|Models]] · [[Conceitos/Views|Views]]

---

## Frontend

Templates HTML renderizados pelo próprio Django, estilizados com Bootstrap. Sem framework JS.

→ [[Estudos/HTML|HTML]] · [[Estudos/CSS|CSS]]

---

## Banco de dados

SQLite no ambiente atual, com plano de migrar para PostgreSQL.

→ [[Projetos/Conecta Senai/Banco de Dados|Banco de Dados]] · [[Estudos/Banco de Dados|PostgreSQL]]

---

## Organização de pastas

- portal/
- templates/
- static/

## Veja também

- [[Sobre]]
- [[Fluxo da Aplicação]]
- [[Roadmap]]
