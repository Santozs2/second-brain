---
type: concept
area: Conceitos
difficulty: beginner
id: cs-websocket
category: Networking
tags:
  - networking
created: 2026-07-05
updated: 2026-07-05
---
# 🔌 WebSocket

---

## 🎯 Conceito

HTTP upgrade → TCP full-duplex  
Servidor pode enviar sem request  
Ideal para real-time

---

## 💻 Uso

```python
from websockets import serve

async def echo(websocket, path):
    async for message in websocket:
        await websocket.send(message)

serve(echo, "localhost", 8765)
```

---

## 🎯 Aplicações

- Chat real-time
- Notificações
- Colaborativo (Google Docs)
- Gaming

---

**Status:** ✅ Completo
