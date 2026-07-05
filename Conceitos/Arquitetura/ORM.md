---
type: concept
tags:
  - concept
created: 2026-06-30
updated: 2026-06-30
status: estavel
---

# ORM

## Definição

Object-Relational Mapping: técnica que mapeia tabelas do banco de dados para classes e objetos da linguagem de programação, evitando escrever SQL manualmente na maior parte dos casos.

## Quando usar

Na maior parte das operações de [[CRUD|CRUD]] em aplicações Django — o ORM gera o SQL por trás dos panos a partir dos [[Models|Models]].

## Boas práticas

- Evitar consultas N+1 (usar `select_related` / `prefetch_related` no Django)
- Usar o ORM para a maioria dos casos, e SQL puro só quando necessário para performance

## Conceitos relacionados

- [[Models|Models]]
- [[Migrations|Migrations]]
- [[CRUD|CRUD]]

## Veja também

- [[Documentações|Documentações]]
