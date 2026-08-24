---
type: concept
area: Conceitos
status: estavel
tags:
  - concept
created: 2026-06-30
updated: 2026-08-24
---
# REST API

## Definição

Estilo arquitetural para APIs HTTP baseado em recursos, verbos HTTP (`GET`, `POST`, `PUT`, `DELETE`) e representações — geralmente JSON.

Formulado por **Roy Fielding (2000)** na tese que também descreve a arquitetura da web. REST não é um protocolo nem um padrão: é um conjunto de **restrições arquiteturais** que, quando respeitadas, produzem sistemas escaláveis e evolutíveis.

## As restrições de Fielding

| Restrição | O que exige |
|---|---|
| **Cliente-servidor** | Separação entre interface e armazenamento |
| **Sem estado** | Cada requisição carrega tudo que precisa; o servidor não guarda sessão |
| **Cacheável** | A resposta declara se pode ser cacheada |
| **Interface uniforme** | Recursos identificados por URI, manipulados por representações |
| **Sistema em camadas** | O cliente não sabe se fala com o servidor final ou com um proxy |
| **Código sob demanda** *(opcional)* | O servidor pode enviar código executável |

> [!note] "Sem estado" é a restrição que mais se viola sem perceber
> Guardar contexto de sessão no servidor entre requisições quebra a escalabilidade horizontal — a próxima requisição pode cair em outro processo que não tem aquele estado. É por isso que [[JWT|JWT]] combina com REST e sessão em memória não combina.

## Quando usar

Para expor dados ou operações de um backend para um frontend ou outro serviço consumir.

**Quando não usar:** quando o cliente precisa de muitos recursos relacionados numa só ida (considere GraphQL), quando a comunicação é bidirecional e contínua ([[cs-websocket|WebSocket]]), ou quando o servidor precisa avisar o cliente de algo ([[Webhooks|webhooks]]).

## Modelagem de recursos

```
GET    /api/cursos/              lista
POST   /api/cursos/              cria
GET    /api/cursos/42/           detalha
PUT    /api/cursos/42/           substitui inteiro
PATCH  /api/cursos/42/           atualiza parcial
DELETE /api/cursos/42/           remove

GET    /api/cursos/42/turmas/    sub-recurso
```

**Recursos são substantivos, não verbos.** A ação está no método HTTP.

```
❌ POST /api/criarCurso
❌ GET  /api/getCursoPorId?id=42
✅ POST /api/cursos/
✅ GET  /api/cursos/42/
```

> [!tip] Ação que não é CRUD vira sub-recurso ou recurso próprio
> `POST /api/fluxos/7/ativar/` é aceitável e comum — melhor do que forçar a ação em um `PATCH` com um campo mágico. O DRF suporta isso com `@action(detail=True)`.

## Status codes que importam

| Código | Significado | Uso típico |
|---|---|---|
| `200` | OK | GET, PUT, PATCH bem-sucedidos |
| `201` | Created | POST que criou recurso (com header `Location`) |
| `204` | No Content | DELETE bem-sucedido |
| `400` | Bad Request | Payload inválido |
| `401` | Unauthorized | Não autenticado (não sabemos quem é você) |
| `403` | Forbidden | Autenticado, mas sem permissão |
| `404` | Not Found | Recurso inexistente |
| `409` | Conflict | Violação de estado (duplicata) |
| `422` | Unprocessable | Sintaxe ok, semântica inválida |
| `429` | Too Many Requests | Rate limit |
| `500` | Server Error | Erro não tratado — **nunca** deveria vazar detalhe |

> [!warning] `401` e `403` significam coisas diferentes
> `401` = "quem é você?" · `403` = "sei quem você é, e você não pode". Em sistema multi-inquilino, prefira `404` a `403` para recurso de outro tenant — `403` confirma que o recurso existe, o que é vazamento de informação.

## Boas práticas

- Nomear recursos no plural (`/cursos`, não `/curso`)
- Usar o verbo HTTP correto para cada ação
- **Paginar listas grandes** — sempre; endpoint sem paginação é bomba-relógio
- Versionar a API (`/api/v1/...`)
- Filtro, ordenação e busca por query string: `?area=mecanica&ordering=-criado_em`
- Erros com corpo estruturado e consistente, nunca só o status
- Documentar com OpenAPI/Swagger (`drf-spectacular`)

```json
// Resposta de erro consistente
{
  "detail": "Dados inválidos.",
  "errors": { "peso": ["Deve estar entre 0 e 5."] }
}
```

## Idempotência

| Método | Idempotente | Seguro (não altera) |
|---|:---:|:---:|
| GET | ✅ | ✅ |
| PUT | ✅ | ❌ |
| DELETE | ✅ | ❌ |
| PATCH | 🟡 depende | ❌ |
| POST | ❌ | ❌ |

Idempotente = repetir a chamada produz o mesmo estado final. Importa porque clientes reenviam em caso de timeout.

## Conceitos relacionados

- [[HTTP|HTTP]] · [[CRUD|CRUD]] · [[Serializers|Serializers]] · [[JWT|JWT]] · [[Views|Views]]
- [[Webhooks|🪝 Webhooks]] · [[cs-websocket|🔌 WebSocket]] · [[Cache|Cache]]
- [[arq-camadas|🏛️ Arquitetura em camadas]]

## Veja também

- [[Documentações|Documentações]]
- [[Django REST Framework|DRF]]
