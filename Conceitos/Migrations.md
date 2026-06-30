---
type: concept
tags:
  - concept
created: 2026-06-30
updated: 2026-06-30
status: estavel
---

# Migrations

## Definição

Histórico versionado de alterações no esquema do banco de dados, gerado automaticamente a partir das mudanças nos [[Conceitos/Models|Models]].

## Quando usar

Sempre que um Model muda (novo campo, nova tabela, relacionamento), gera-se uma migration e ela é aplicada ao banco — em vez de alterar tabelas manualmente.

## Boas práticas

- Nunca editar uma migration já aplicada em produção
- Revisar a migration gerada antes de aplicar

## Conceitos relacionados

- [[Conceitos/Models|Models]]
- [[Conceitos/ORM|ORM]]

## Veja também

- [[Recursos/Documentações|Documentações]]
