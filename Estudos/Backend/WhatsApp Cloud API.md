---
type: tech
area: Estudos
status: aprendendo
tecnologia: WhatsApp Cloud API
tags:
  - tech
  - estudo
  - backend
  - api
created: 2026-08-24
updated: 2026-08-24
---
# 💬 WhatsApp Cloud API

> [!tip] Status
> 🟢 Em uso

## 📝 Resumo

API oficial da Meta para enviar e receber mensagens de WhatsApp programaticamente, hospedada pela própria Meta (diferente da *On-Premises*, que exige infraestrutura própria).

É a alternativa **oficial** às bibliotecas não-oficiais que automatizam o WhatsApp Web — essas violam os termos de uso e resultam em banimento do número.

## 🧱 A hierarquia de objetos

```
Meta Business Account
└── WhatsApp Business Account (WABA)
    └── Número de telefone (phone_number_id)
        └── Mensagens
```

Você precisa de: conta Business verificada, WABA, número registrado, token de acesso e um App na Meta.

> [!warning] A verificação do negócio demora e é o gargalo do cronograma
> A verificação pode levar de dias a semanas, e exige documentação da empresa. **Comece esse processo antes de escrever código** — é a única parte do projeto que não depende de você. Enquanto isso, o número de teste que a Meta fornece permite desenvolver.

## 📤 Enviar mensagem

```python
import requests

def enviar_texto(phone_number_id: str, token: str, para: str, texto: str) -> dict:
    resp = requests.post(
        f"https://graph.facebook.com/v21.0/{phone_number_id}/messages",
        headers={"Authorization": f"Bearer {token}"},
        json={
            "messaging_product": "whatsapp",
            "to": para,                       # E.164, sem "+": 5511999999999
            "type": "text",
            "text": {"body": texto},
        },
        timeout=10,
    )
    resp.raise_for_status()
    return resp.json()
```

## 📥 Receber: webhook

A Meta faz `POST` na sua URL a cada evento. Ver [[Webhooks|🪝 Webhooks]].

```python
# Verificação inicial (GET) — a Meta chama uma vez ao configurar
def verificar(request):
    if request.GET.get("hub.verify_token") == settings.VERIFY_TOKEN:
        return HttpResponse(request.GET.get("hub.challenge"))
    return HttpResponse(status=403)
```

O payload é profundamente aninhado:

```
entry[0].changes[0].value.messages[0]     ← mensagens recebidas
entry[0].changes[0].value.statuses[0]     ← status de entrega/leitura
```

> [!tip] Persista o payload cru antes de tentar interpretá-lo
> A estrutura tem muitos níveis opcionais e varia por tipo de evento. Salvar o JSON bruto e processar depois permite reprocessar quando você descobrir um caso não tratado — em vez de perder o evento.

## ⏰ A janela de 24 horas

A regra que define a arquitetura de qualquer sistema sobre esta API:

| Situação | O que pode enviar |
|---|---|
| **Dentro** de 24h da última mensagem do contato | Texto livre, mídia, qualquer coisa |
| **Fora** da janela | **Somente template aprovado** pela Meta |

```python
def dentro_da_janela(conversa) -> bool:
    ultima = conversa.ultima_mensagem_recebida_em
    return ultima and (timezone.now() - ultima) < timedelta(hours=24)
```

> [!important] Templates precisam de aprovação prévia e podem ser recusados
> Cada template é submetido à Meta e revisado (de minutos a dias). Conteúdo promocional é recusado com frequência. **Planeje os templates cedo** — descobrir na véspera que o template foi negado trava a funcionalidade inteira.

Um bot que só **reage** a mensagens recebidas está sempre dentro da janela — o que simplifica bastante o desenho.

## 💰 Cobrança

A cobrança é por **conversa** de 24 horas, não por mensagem, com preço variável por categoria (marketing, utilidade, autenticação, serviço) e por país. Conversas iniciadas pelo usuário costumam ser mais baratas que as iniciadas pela empresa.

## 🔐 Segurança

- Verificar a assinatura `X-Hub-Signature-256` em todo webhook
- Token de acesso **nunca** no código; use variável de ambiente
- Preferir token de sistema de longa duração ao token temporário do painel
- HTTPS obrigatório no endpoint

## ⚠️ Limites de taxa

Existe um limite de mensagens iniciadas por empresa em 24h (tiers de 1k, 10k, 100k, ilimitado), que sobe conforme qualidade e volume. A **classificação de qualidade** do número cai com bloqueios e denúncias — e um número de baixa qualidade tem o limite reduzido.

## 🔗 Conceitos relacionados

- [[Webhooks|🪝 Webhooks]] · [[REST API|REST API]] · [[HTTP|HTTP]]
- [[Django Channels|🔌 Django Channels]] · [[arq-event-driven|📡 Eventos]]

## Veja também

- [[Estudos|📚 Estudos]]
- [[Documentações|Documentações]]
