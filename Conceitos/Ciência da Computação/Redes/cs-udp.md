---
type: concept
area: Conceitos
difficulty: beginner
id: cs-udp
category: Networking
tags:
  - networking
created: 2026-07-05
updated: 2026-07-05
---
# 📡 UDP (User Datagram Protocol)

---

## 🎯 Características

```
Connectionless - sem handshake
Unreliable - sem garantia entrega
Rápido - sem overhead
Stateless
```

---

## 📦 Datagram

```
Header: Source port, Dest port, Length, Checksum
Payload: Dados
```

---

## vs TCP

```
TCP: Confiável, lento, conexão
UDP: Rápido, não confiável, sem conexão
```

---

## 🎯 Uso

- DNS queries
- VOIP
- Streaming (perder 1 frame OK)
- Gaming (baixa latência > garantia)

---

**Status:** ✅ Completo

## 🔗 Ver também nesta área

- [[cs-dns]]
- [[cs-http]]
- [[cs-load-balancing]]
