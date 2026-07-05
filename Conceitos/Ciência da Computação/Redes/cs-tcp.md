---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-tcp
category: Networking
tags:
  - networking
created: 2026-07-05
updated: 2026-07-05
---
# 🌐 TCP (Transmission Control Protocol)

> Protocolo confiável, orientado a conexão.

---

## 🤝 Three-Way Handshake

```
Client                        Server
  |                             |
  |--- SYN seq=x ------------->|
  |<---- SYN-ACK seq=y, ack=x+1|
  |--- ACK seq=x+1, ack=y+1 -->|
  |                             |
  Connected!
```

---

## 📊 Confiabilidade

**Sequencing:** Números de sequência garantem ordem  
**ACK:** Confirmação de recebimento  
**Retransmission:** Timeout → resend  
**Checksums:** Detecção de erro

---

## 🎯 Congestion Control

```
Slow Start → Congestion Avoidance → Fast Recovery
Aumenta window até perder pacote
```

---

## 🚪 Fechamento

```
Client                        Server
  |--- FIN seq=x ------------->|
  |<---- ACK seq=y -----------  |
  |<---- FIN seq=z -----------  |
  |--- ACK seq=z+1 ----------->|
```

---

## vs UDP

TCP: Confiável, mais lento  
UDP: Rápido, sem garantia

---

**Status:** ✅ Completo
