---
type: concept
area: Conceitos
status: estavel
tags:
  - concept
created: 2026-06-30
updated: 2026-06-30
---
# REST API

## Definição

Estilo arquitetural para APIs HTTP baseado em recursos, verbos HTTP (`GET`, `POST`, `PUT`, `DELETE`) e representações — geralmente JSON.

## Quando usar

Para expor dados ou operações de um backend para um frontend ou outro serviço consumir.

## Boas práticas

- Nomear recursos no plural (`/cursos`, não `/curso`)
- Usar o verbo HTTP correto para cada ação
- Paginar listas grandes
- Versionar a API (`/api/v1/...`)

## Conceitos relacionados

- [[HTTP|HTTP]]
- [[CRUD|CRUD]]
- [[Serializers|Serializers]]
- [[JWT|JWT]]

## Veja também

- [[Documentações|Documentações]]
