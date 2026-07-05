---
type: concept
id: cs-aes
created: 2026-07-05
updated: 2026-07-05
category: Security
tags:
  - type/concept
  - domain/security
  - difficulty/advanced
---

# 🔐 AES (Advanced Encryption Standard)

> Criptografia simétrica padrão.

---

## 🔑 Tamanhos

```
AES-128: 128 bits (10 rounds)
AES-192: 192 bits (12 rounds)
AES-256: 256 bits (14 rounds)
```

---

## 🔄 Processo

```
1. Subtituição (S-box)
2. Permutação (ShiftRows)
3. Mistura (MixColumns)
4. Add Round Key
(repete 10-14 rounds)
```

---

## 💪 Segurança

AES-256 é praticamente seguro  
Nenhum ataque conhecido (2024)

---

## 🎯 Uso

- Criptografia de dados
- Padrão NIST
- TLS 1.2+
- Disco encryption

---

**Status:** ✅ Completo
