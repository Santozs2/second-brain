---
type: concept
id: cs-hash
created: 2026-07-05
updated: 2026-07-05
category: Security
tags:
  - type/concept
  - domain/security
  - difficulty/beginner
---

# #️⃣ Hash Functions

> Transformar dados → número fixo. Impossível reverter.

---

## 🎯 Propriedades

1. **Determinístico:** Mesmo input → mesmo hash
2. **Rápido:** O(n) computar hash
3. **Avalanche:** Pequena mudança → hash diferente
4. **Collision Resistant:** Difícil encontrar mesma saída
5. **Irreversível:** Não volta ao original

---

## 📊 Algoritmos

```
MD5      - 128 bits (quebrado, não usar)
SHA-1    - 160 bits (fraco)
SHA-256  - 256 bits (seguro)
SHA-3    - Novo (mais seguro)
```

---

## 🔒 Salting

```python
password = "mypassword"
salt = random_bytes(16)
hash = SHA256(password + salt)

# Mesmo mesmo padrão não gera colisão
```

---

## 🎯 Uso

✅ **Senhas:** Salt + hash  
✅ **Integridade:** Detectar modificação  
✅ **Blockchain:** Proof of work  
✅ **Tabelas hash:** Distribuir dados

---

## ⚠️ Bcrypt/Argon2

Mais lento que SHA (requer 100ms)  
Melhor para senhas (resiste brute force)

---

**Status:** ✅ Completo
