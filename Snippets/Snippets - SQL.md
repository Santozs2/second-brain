---
type: snippet
tags:
  - snippet
linguagem: SQL
created: 2026-06-30
updated: 2026-06-30
status: estavel
---

# 🧩 Snippets de SQL

> [!tip] Como usar
> Cada `###` abaixo é um snippet independente. Adicione novos no final usando o mesmo padrão.

## JOIN com contagem

```sql
SELECT u.nome, COUNT(p.id) AS total
FROM usuarios u
LEFT JOIN pedidos p ON p.usuario_id = u.id
GROUP BY u.nome;
```

**Quando usar:** relacionar duas tabelas e contar registros relacionados (ex: pedidos por usuário).

#snippet #sql

---

## Evitar duplicados

```sql
SELECT DISTINCT cidade FROM clientes;
```

**Quando usar:** quando precisar de valores únicos de uma coluna.

#snippet #sql

---

## Backup e restore (PostgreSQL via Docker)

```bash
docker exec -t meu_banco pg_dump -U postgres meubanco > backup.sql
docker exec -i meu_banco psql -U postgres meubanco < backup.sql
```

**Quando usar:** salvar ou restaurar o banco de dados de um container Docker.

#snippet #sql #docker

## Veja também

- [[Banco de Dados|Banco de Dados]]
- [[Snippets|Todos os snippets]]
