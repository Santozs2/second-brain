---
type: concept
area: Conceitos
status: estavel
difficulty: advanced
id: arq-event-driven
category: Arquitetura
tags:
  - arquitetura
  - concept
created: 2026-08-24
updated: 2026-08-24
---
# 📡 Arquitetura Orientada a Eventos

> Em vez de A chamar B, A **anuncia que algo aconteceu** e quem se interessar reage. Desacopla quem produz de quem consome.

---

## 📖 Os dois modelos

```
SÍNCRONO (comando)              ASSÍNCRONO (evento)
A ──chama──► B                  A ──publica──► [barramento] ──► B
   espera                                              └──► C
   resposta                                            └──► D

A sabe quem é B                 A não sabe quem escuta
A depende de B estar de pé      A não depende de ninguém
```

---

## 🧱 Componentes

| Componente | Papel |
|---|---|
| **Produtor** | Publica o evento; não sabe quem consome |
| **Evento** | Fato que já aconteceu, no passado: `PedidoCriado` |
| **Broker** | Transporta e retém: RabbitMQ, Kafka, Redis |
| **Consumidor** | Reage ao evento |

> [!important] Evento descreve o passado; comando pede o futuro
> `EnviarEmail` é um **comando** — tem um destinatário esperado e pode falhar de forma significativa. `UsuárioCadastrado` é um **evento** — é um fato consumado, e ter zero ou cinco consumidores não muda nada para quem publicou. Confundir os dois produz sistemas que parecem desacoplados e não são.

---

## 🐍 No ecossistema Django

### Signals — eventos dentro do processo

```python
from django.dispatch import Signal, receiver

quiz_finalizado = Signal()   # providing_args: attempt

# Produtor
quiz_finalizado.send(sender=QuizAttempt, attempt=attempt)

# Consumidores independentes
@receiver(quiz_finalizado)
def registrar_metrica(sender, attempt, **kwargs):
    Metrica.objects.create(tipo="quiz", ref=attempt.id)

@receiver(quiz_finalizado)
def agendar_entrega_ia(sender, attempt, **kwargs):
    gerar_entrega.delay(attempt.id)
```

> [!warning] Signals são síncronos e escondem o fluxo
> Um `receiver` roda **dentro** da mesma requisição: se ele demora, a resposta demora; se ele levanta exceção, quebra o fluxo do produtor. Além disso, eles tornam o código difícil de seguir — nada no ponto de emissão indica quem vai executar. **Para lógica que pertence ao fluxo principal, prefira chamada explícita.**

### Celery — processamento assíncrono de verdade

```python
from celery import shared_task

@shared_task(bind=True, max_retries=3, default_retry_delay=60)
def gerar_entrega(self, attempt_id: int):
    try:
        entregar(attempt_id)
    except ProviderError as exc:
        raise self.retry(exc=exc)        # backoff automático

# Enfileira e devolve na hora
gerar_entrega.delay(attempt.id)
```

Isto é o padrão correto para tudo que é **lento, externo ou pode falhar**: chamada de LLM, envio de e-mail, geração de relatório, processamento de mídia.

---

## ✅ Quando compensa

| Caso | Por quê |
|---|---|
| Trabalho lento fora da requisição | O usuário não espera |
| Integração com serviço instável | Retry automático |
| Vários efeitos para um mesmo fato | Consumidores independentes |
| Picos de carga | A fila absorve |
| Auditoria e histórico | O log de eventos é o registro |

## ❌ Quando atrapalha

| Caso | Por quê |
|---|---|
| O resultado é necessário agora | Assíncrono não ajuda |
| Fluxo simples e rápido | Complexidade sem retorno |
| Precisa de transação forte | Consistência eventual não serve |
| Ordem estrita é obrigatória | Difícil de garantir entre consumidores |

---

## ⚠️ O que muda quando você vai para eventos

### Consistência eventual
O sistema fica temporariamente inconsistente. A interface precisa refletir isso — "processando", não um número que ainda não existe.

### Entrega ao menos uma vez
A maioria dos brokers pode entregar o mesmo evento duas vezes. **Consumidores precisam ser idempotentes.**

```python
@shared_task
def gerar_entrega(attempt_id: int):
    # Idempotência: processar duas vezes tem o mesmo efeito que uma
    if Entrega.objects.filter(attempt_id=attempt_id).exists():
        return
    ...
```

### Depuração distribuída
Sem `trace_id` propagado, investigar uma falha vira arqueologia. Instrumente desde o início.

### Dead letter queue
Eventos que falham repetidamente precisam de um destino, ou somem silenciosamente.

---

## 📚 Referências

- **Hohpe & Woolf (2003)** — *Enterprise Integration Patterns* — o catálogo de referência
- **Kleppmann, M. (2017)** — *Designing Data-Intensive Applications*, cap. 11
- **Fowler, M.** — *What do you mean by "Event-Driven"?*
- Documentação do **Celery** e do **Django Signals**

---

## 🔗 Conceitos relacionados

- [[arq-monolito-vs-microservicos|🧊 Monólito × Microsserviços]] · [[arq-design-patterns|🎨 Observer]]
- [[Cache|Cache/Redis]] · [[cs-websocket|🔌 WebSocket]]
- [[dad-transacoes-acid|🔒 Transações e ACID]]

## Veja também

- [[Conceitos|🧠 Conceitos]]

---

**Status:** ✅ Completo
