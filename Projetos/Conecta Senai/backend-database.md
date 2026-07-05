---
type: project
area: Projetos
status: em-andamento
tags:
  - project
created: 2026-06-30
updated: 2026-06-30
---
# Banco de Dados — Conecta SENAI

```mermaid
erDiagram

CURSO ||--o{ INTERESSE : possui

CURSO {

int id

string nome

string modalidade

}

INTERESSE {

int id

string nome

string email

string telefone

}
```

---

## Entidades

### Curso

Representa um curso disponível.

---

### Interesse

Representa um usuário interessado.

## Veja também

- [[Arquitetura]]
- [[Banco de Dados|Banco de Dados (conceito)]]
- [[Migrations|Migrations]]
