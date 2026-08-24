---
type: concept
area: Conceitos
status: estavel
difficulty: intermediate
id: webhooks
category: API
tags:
  - concept
  - api
created: 2026-08-24
updated: 2026-08-24
---
# 🪝 Webhooks

> Uma API invertida: em vez de você perguntar "aconteceu alguma coisa?", o serviço avisa você. É *push*, não *pull*.

---

## 📖 Polling versus webhook

```
POLLING                          WEBHOOK
você ──"tem novidade?"──► API    serviço ──POST──► seu endpoint
     ◄──── "não" ────                    (só quando acontece)
     (repete a cada 5s)
```

| | Polling | Webhook |
|---|---|---|
| Latência | Até o intervalo | Imediata |
| Requisições desperdiçadas | Muitas | Nenhuma |
| Complexidade | Baixa | Média |
| Precisa de URL pública | ❌ | ✅ |

---

## 🔄 O fluxo

```
1. Você registra uma URL no serviço
2. Acontece o evento
3. O serviço faz POST na sua URL com o payload
4. Você responde 200 rápido
5. Se não responder 200, o serviço tenta de novo
```

---

## ⚡ A regra de ouro: responda 200 imediatamente

```python
# ❌ Processa antes de responder — o provedor dá timeout e reenvia
@csrf_exempt
def webhook(request):
    payload = json.loads(request.body)
    processar_tudo(payload)          # 8 segundos
    return HttpResponse(status=200)

# ✅ Valida, persiste o cru, enfileira, responde
@csrf_exempt
def webhook(request):
    if not assinatura_valida(request):
        return HttpResponse(status=403)
    evento = EventoBruto.objects.create(payload=request.body.decode())
    processar_evento.delay(evento.id)      # assíncrono
    return HttpResponse(status=200)        # < 100ms
```

> [!important] Timeout do seu lado vira reentrega do lado deles
> Provedores costumam desistir entre 5 e 10 segundos e reenviar o evento. Se o seu processamento é lento, você entra em um laço de reentregas — e processa o mesmo evento várias vezes. **Persistir o payload cru e processar em fila resolve os dois problemas de uma vez.**

---

## 🔐 Verificação de assinatura

Sua URL é pública: qualquer um pode fazer POST nela. A assinatura prova que o payload veio de quem diz ter vindo.

```python
import hmac, hashlib

def assinatura_valida(request) -> bool:
    recebida = request.headers.get("X-Hub-Signature-256", "")
    esperada = "sha256=" + hmac.new(
        settings.WEBHOOK_SECRET.encode(),
        request.body,                      # o corpo CRU, não o JSON parseado
        hashlib.sha256,
    ).hexdigest()
    return hmac.compare_digest(recebida, esperada)
```

> [!warning] Dois detalhes que anulam a proteção se forem ignorados
> **1. Use o corpo bruto.** Serializar o JSON de novo muda espaços e ordem de chaves, e o hash não bate.
> **2. Use `hmac.compare_digest`, nunca `==`.** A comparação normal sai no primeiro byte diferente, o que abre um ataque de temporização.

Ver [[cs-hash|🔐 Hash]] e [[cs-authentication|🔑 Autenticação]].

---

## 🔁 Idempotência

Webhooks têm entrega **ao menos uma vez**. O mesmo evento vai chegar duplicado em algum momento.

```python
@shared_task
def processar_evento(evento_id: int):
    evento = EventoBruto.objects.get(pk=evento_id)
    dados = json.loads(evento.payload)

    # Chave de deduplicação vinda do provedor
    _, criado = EventoProcessado.objects.get_or_create(
        external_id=dados["id"],
        defaults={"tipo": dados["type"]},
    )
    if not criado:
        return          # já processado; sai sem efeito colateral
    aplicar(dados)
```

---

## ✅ Checklist de implementação

- [ ] Endpoint isento de CSRF (não vem de formulário seu)
- [ ] Assinatura verificada **antes** de qualquer processamento
- [ ] Payload cru persistido antes de processar
- [ ] Processamento assíncrono; resposta 200 rápida
- [ ] Idempotência por identificador do provedor
- [ ] HTTPS obrigatório
- [ ] Log de tudo que chega, inclusive o que for rejeitado
- [ ] Endpoint de verificação (GET) se o provedor exigir

---

## 🛠️ Testar em desenvolvimento

O serviço precisa alcançar sua máquina. Ferramentas de túnel:

```bash
ngrok http 8000            # gera uma URL pública temporária
cloudflared tunnel --url http://localhost:8000
```

---

## 🔗 Conceitos relacionados

- [[REST API|REST API]] · [[HTTP|HTTP]] · [[cs-hash|🔐 Hash]]
- [[arq-event-driven|📡 Arquitetura orientada a eventos]] · [[cs-websocket|🔌 WebSocket]]
- [[cs-authentication|🔑 Autenticação]]

## Veja também

- [[Conceitos|🧠 Conceitos]]
- [[Documentações|Documentações]]

---

**Status:** ✅ Completo
