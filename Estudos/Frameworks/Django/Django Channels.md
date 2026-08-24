---
type: tech
area: Estudos
status: aprendendo
tecnologia: Django Channels
tags:
  - tech
  - estudo
  - backend
  - websocket
created: 2026-08-24
updated: 2026-08-24
---
# 🔌 Django Channels

> [!tip] Status
> 🟢 Em uso

## 📝 Resumo

Extensão que leva o [[Django|Django]] além do ciclo requisição-resposta do HTTP, adicionando suporte a **WebSocket**, tarefas de longa duração e protocolos assíncronos — via **ASGI** em vez de WSGI.

```
WSGI (Django puro)      ASGI (Channels)
requisição → resposta   conexão persistente, bidirecional
sem estado              mantém a conexão aberta
```

## 🧠 Conceitos principais

- **ASGI** — a interface assíncrona que substitui o WSGI; permite conexões de longa duração
- **Consumer** — o equivalente da view para WebSocket: trata `connect`, `receive` e `disconnect`
- **Channel layer** — o barramento que permite a um processo enviar mensagem a conexões de outro processo (normalmente Redis)
- **Group** — um conjunto nomeado de conexões; publicar no grupo alcança todos os membros
- **Routing** — o `urls.py` do WebSocket

> [!important] Sem channel layer, cada processo fica isolado
> Em produção existem vários workers. Uma mensagem que chega no worker 1 precisa alcançar um usuário conectado no worker 3 — e só o channel layer (Redis) faz essa ponte. Em desenvolvimento a camada em memória funciona; **em produção ela quebra silenciosamente**, entregando mensagem só para parte dos usuários.

## 💻 Estrutura mínima

```python
# consumers.py
from channels.generic.websocket import AsyncJsonWebsocketConsumer

class InboxConsumer(AsyncJsonWebsocketConsumer):

    async def connect(self):
        user = self.scope["user"]
        if not user.is_authenticated:
            return await self.close(code=4001)
        self.group = f"org_{user.org_id}"
        await self.channel_layer.group_add(self.group, self.channel_name)
        await self.accept()

    async def disconnect(self, code):
        await self.channel_layer.group_discard(self.group, self.channel_name)

    async def mensagem_nova(self, event):
        """Chamado quando alguém publica type='mensagem.nova' no grupo."""
        await self.send_json(event["payload"])
```

```python
# routing.py
websocket_urlpatterns = [path("ws/inbox/", InboxConsumer.as_asgi())]
```

```python
# settings.py
ASGI_APPLICATION = "config.asgi.application"
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels_redis.core.RedisChannelLayer",
        "CONFIG": {"hosts": [("127.0.0.1", 6379)]},
    }
}
```

## 📡 Publicar a partir de código síncrono

```python
from asgiref.sync import async_to_sync
from channels.layers import get_channel_layer

def notificar(org_id: int, payload: dict):
    async_to_sync(get_channel_layer().group_send)(
        f"org_{org_id}",
        {"type": "mensagem.nova", "payload": payload},   # type → método do consumer
    )
```

> [!note] O `type` do evento vira o nome do método
> `{"type": "mensagem.nova"}` chama `async def mensagem_nova(self, event)`. O ponto vira underscore. Errar esse mapeamento produz uma falha silenciosa — a mensagem é publicada e ninguém a recebe.

## 🔐 Autenticação em WebSocket

O navegador **não permite** definir cabeçalhos no handshake do WebSocket — então `Authorization: Bearer` não é uma opção.

```js
const ws = new WebSocket(`ws://localhost:8000/ws/inbox/?token=${token}`);
```

O token vai na query string e é validado por um middleware ASGI customizado.

> [!warning] Token em query string aparece em log de servidor
> É a solução prática e amplamente usada, mas tem esse custo. Mitigações: usar token de curta duração emitido só para a conexão WebSocket, e garantir `wss://` (TLS) em produção, que ao menos protege o trânsito.

## 🗄️ ORM dentro de código assíncrono

```python
from channels.db import database_sync_to_async

@database_sync_to_async
def buscar_conversa(pk):
    return Conversation.objects.select_related("contact").get(pk=pk)
```

Chamar o ORM síncrono direto em um consumer assíncrono levanta `SynchronousOnlyOperation`.

## 🚀 Produção

```bash
daphne -b 0.0.0.0 -p 8000 config.asgi:application
# ou
uvicorn config.asgi:application --host 0.0.0.0 --port 8000
```

`runserver` serve ASGI em desenvolvimento, mas não em produção. O proxy reverso (Nginx) precisa encaminhar os cabeçalhos de *upgrade*.

## 🔗 Conceitos relacionados

- [[Django|Django]] · [[cs-websocket|🔌 WebSocket]] · [[Cache|Redis]]
- [[JWT|JWT]] · [[cs-authentication|🔑 Autenticação]] · [[arq-event-driven|📡 Eventos]]

## Veja também

- [[Estudos|📚 Estudos]]
- [[Documentações|Documentações]]
