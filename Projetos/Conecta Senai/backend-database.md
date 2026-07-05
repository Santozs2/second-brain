---
type: project
tags:
  - project
created: 2026-06-30
updated: 2026-06-30
status: em-andamento
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
- [[Estudos/Banco de Dados|Banco de Dados (conceito)]]
- [[Conceitos/Migrations|Migrations]]
