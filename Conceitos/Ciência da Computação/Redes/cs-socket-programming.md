---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-socket-programming
category: Networking
tags:
  - networking
created: 2026-07-05
updated: 2026-07-05
---
# 🔌 Socket Programming

---

## 📝 Server Example

```python
import socket

# TCP Server
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(('localhost', 5000))
server.listen(1)

while True:
    client, addr = server.accept()
    data = client.recv(1024)
    client.send(b'Hello!')
    client.close()
```

---

## 📝 Client Example

```python
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('localhost', 5000))
client.send(b'Hello Server!')
response = client.recv(1024)
client.close()
```

---

## 🎯 UDP Socket

```python
server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server.bind(('localhost', 5000))
data, addr = server.recvfrom(1024)
server.sendto(b'Response', addr)
```

---

**Status:** ✅ Completo
