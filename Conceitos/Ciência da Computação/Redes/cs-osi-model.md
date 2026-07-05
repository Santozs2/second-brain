---
type: concept
area: Conceitos
difficulty: beginner
id: cs-osi-model
category: Networking
tags:
  - networking
created: 2026-07-05
updated: 2026-07-05
---
# 🌐 OSI Model

> 7-layer network model (conceptual).

---

## 🔢 7 Camadas

```
7. Application   - HTTP, DNS, FTP
6. Presentation  - Criptografia, Compressão
5. Session       - Gerência de conexão
4. Transport     - TCP, UDP
3. Network       - IP, Roteamento
2. Data Link     - Ethernet, MAC
1. Physical      - Cabos, Sinais elétricos
```

---

## 📦 Encapsulation

```
App: Dados
T4: Segment (dados + porta)
N3: Packet (segment + IP)
L2: Frame (packet + MAC)
P1: Bits
```

---

## vs TCP/IP

OSI: Conceitual (7 layers)  
TCP/IP: Prático (4 layers)

---

**Status:** ✅ Completo
