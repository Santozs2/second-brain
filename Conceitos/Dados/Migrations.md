---
type: concept
area: Conceitos
status: estavel
tags:
  - concept
created: 2026-06-30
updated: 2026-06-30
---
# Migrations

## Definição

Histórico versionado de alterações no esquema do banco de dados, gerado automaticamente a partir das mudanças nos [[Models|Models]].

## Quando usar

Sempre que um Model muda (novo campo, nova tabela, relacionamento), gera-se uma migration e ela é aplicada ao banco — em vez de alterar tabelas manualmente.

## Boas práticas

- Nunca editar uma migration já aplicada em produção
- Revisar a migration gerada antes de aplicar

## Conceitos relacionados

- [[Models|Models]]
- [[ORM|ORM]]

## Veja também

- [[Documentações|Documentações]]
