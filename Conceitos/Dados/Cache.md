---
type: concept
area: Conceitos
status: estavel
tags:
  - concept
created: 2026-06-30
updated: 2026-08-24
---
# Cache

## Definição

Armazenamento temporário de dados já processados, para evitar reprocessar ou reconsultar algo caro toda vez.

O cache aposta na **localidade**: o que foi pedido agora tende a ser pedido de novo em breve.

## Quando usar

Em respostas de [[HTTP|HTTP]]/API que mudam pouco, ou em consultas ao banco que se repetem muito.

**O candidato ideal:** caro de produzir, lido muitas vezes, escrito poucas vezes, e tolerante a estar levemente desatualizado.

## As camadas de cache

```
Navegador  →  CDN  →  Proxy reverso  →  Aplicação  →  Banco
   ↑           ↑           ↑                ↑           ↑
 local      borda      Nginx          Redis/memória  buffer pool
```

Cada camada evita trabalho da seguinte. Quanto mais perto do usuário, mais barato.

## Estratégias

| Estratégia | Como funciona | Uso |
|---|---|---|
| **Cache-aside** | A aplicação consulta o cache; se não achar, busca e grava | O padrão |
| **Read-through** | O cache busca sozinho quando não tem | Menos código |
| **Write-through** | Escreve no cache e no banco juntos | Consistência alta |
| **Write-behind** | Escreve no cache, persiste depois | Rápido; risco de perda |

```python
from django.core.cache import cache

def ranking_do_perfil(chave: str):
    resultado = cache.get(chave)              # 1. tenta o cache
    if resultado is None:                     # 2. miss
        resultado = calcular_ranking(chave)   # 3. produz
        cache.set(chave, resultado, 3600)     # 4. guarda com TTL
    return resultado
```

## Invalidação

> Existem duas coisas difíceis em computação: invalidação de cache e nomear coisas. — *Phil Karlton*

| Técnica | Como |
|---|---|
| **TTL** | Expira sozinho depois de N segundos — simples e suficiente na maioria dos casos |
| **Invalidação explícita** | `cache.delete(chave)` quando o dado muda |
| **Versão na chave** | `curso:42:v3` — mudar a versão aposenta o antigo |
| **Chave por conteúdo** | Hash da entrada; conteúdo diferente, chave diferente |

> [!success] Chave derivada do conteúdo dispensa invalidação
> Se a chave é o hash da entrada canônica, uma entrada diferente gera uma chave diferente e o valor antigo simplesmente deixa de ser consultado. Não há o que invalidar.
> ```python
> def chave(payload: dict) -> str:
>     canonico = json.dumps(payload, sort_keys=True, ensure_ascii=False)
>     return "rec:" + hashlib.sha256(canonico.encode()).hexdigest()
> ```

## Os três problemas clássicos

| Problema | O que é | Mitigação |
|---|---|---|
| **Stampede** | A chave expira e mil requisições recalculam ao mesmo tempo | Lock; TTL com jitter aleatório |
| **Penetração** | Consultas a chaves que nunca existem passam direto ao banco | Cachear o "não existe" com TTL curto |
| **Avalanche** | Muitas chaves expiram no mesmo instante | Espalhar os TTLs |

```python
import random
cache.set(chave, valor, 3600 + random.randint(0, 300))   # jitter
```

## Cache HTTP

```
Cache-Control: public, max-age=3600        # cacheável por 1h
Cache-Control: private, no-store           # nunca cachear (dado sensível)
ETag: "abc123"                             # validador de conteúdo
```

Com `ETag`, o cliente envia `If-None-Match` e o servidor pode responder `304 Not Modified` — sem corpo, economizando banda.

## Boas práticas

- **TTL sempre** — cache sem expiração vira dado errado permanente
- **Nunca cachear dado sensível** em camada compartilhada
- **Nunca cachear por usuário em cache global** sem incluir o id do usuário na chave
- Monitorar a **taxa de acerto** — abaixo de ~70%, o cache pode estar custando mais do que ajuda
- O sistema precisa **funcionar com o cache vazio** (e desligado)

> [!warning] Chave sem escopo de usuário é vazamento de dados
> `cache.set("dashboard", dados)` faz o dashboard do primeiro usuário ser servido a todos os outros. Toda chave de dado personalizado precisa do identificador: `f"dashboard:{user.id}"`. Em sistema multi-inquilino, inclua também o tenant.

## Conceitos relacionados

- [[HTTP|HTTP]] · [[dad-otimizacao-consultas|⚡ Otimização de consultas]]
- [[cs-memory-hierarchy|🧠 Hierarquia de memória]] · [[cs-hash-table|🔑 Tabelas hash]]
- [[ia-tokens-e-custo|💰 Tokens e custo]] · [[arq-event-driven|📡 Eventos]]

## Veja também

- [[Documentações|Documentações]]
