---
type: tech
area: Estudos
status: aprendendo
tecnologia: Banco de Dados
tags:
  - tech
  - estudo
  - backend
  - banco-de-dados
created: 2026-06-30
updated: 2026-06-30
---
# Banco de Dados (SQL / PostgreSQL)

> [!tip] Status
> 🟡 Aprendendo

## 📝 Resumo

Bancos de dados relacionais organizam dados em tabelas. SQL é a linguagem usada para consultá-los e manipulá-los. PostgreSQL é um dos bancos relacionais mais usados no mercado.

## 🧠 Conceitos principais

- **Tabelas, colunas e tipos de dados**
- **Chaves primária e estrangeira**
- **Consultas**: `SELECT`, `WHERE`, `JOIN`, `GROUP BY`
- **Manipulação**: `INSERT`, `UPDATE`, `DELETE`
- **Índices** e performance básica
- **Migrations** (via Django ORM)
- **Normalização**

## 💻 Exemplos

```sql
SELECT u.nome, COUNT(p.id) AS total_pedidos
FROM usuarios u
JOIN pedidos p ON p.usuario_id = u.id
GROUP BY u.nome
ORDER BY total_pedidos DESC;
```

```sql
CREATE TABLE tarefas (
  id SERIAL PRIMARY KEY,
  titulo VARCHAR(200) NOT NULL,
  concluida BOOLEAN DEFAULT FALSE
);
```

## 🔗 Links úteis

- [Documentação oficial PostgreSQL](https://www.postgresql.org/docs/)
- [SQLBolt - Aprenda SQL interativamente](https://sqlbolt.com/)

## ✅ Checklist de aprendizado

- [x] SELECT, WHERE, ORDER BY
- [ ] JOINs (INNER, LEFT)
- [ ] Agregações (GROUP BY, HAVING)
- [ ] Índices e performance
- [ ] Modelagem de dados

## 🗒️ Notas pessoais


## 🧩 Conceitos relacionados

- [[ORM|ORM]]
- [[Migrations|Migrations]]

## 🔗 Veja também

- [[Django|Django]]
- [[Docker|Docker]]
- [[Snippets - SQL|Snippets de SQL]]

