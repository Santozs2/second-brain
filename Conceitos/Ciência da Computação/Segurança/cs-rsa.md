---
type: concept
id: cs-rsa
created: 2026-07-05
updated: 2026-07-05
category: Security
tags:
  - type/concept
  - domain/security
  - difficulty/advanced
---

# 🔐 RSA Encryption

> Criptografia assimétrica baseada em números primos.

---

## 🔑 Geração de Chaves

```python
# 1. Escolhe 2 primos grandes: p, q
p = 61
q = 53

# 2. n = p * q (parte pública)
n = 3233

# 3. φ(n) = (p-1)(q-1)
phi = 3120

# 4. Escolhe e tal que gcd(e, φ) = 1
e = 17  # chave pública = (n, e)

# 5. Calcula d = e^(-1) mod φ
d = 2753  # chave privada = (n, d)
```

---

## 🔒 Criptografia

```
Plaintext m
Ciphertext: c = m^e mod n
Decryption: m = c^d mod n
```

---

## 💪 Segurança

Quebrar RSA = fatorizar n em p*q  
Muito difícil para n de 2048 bits

---

## ⚠️ Problemas

- Lento (1000x mais lento que AES)
- Precisa padding (OAEP)
- Não é determinístico (precisa aleatoriedade)

---

## 🎯 Uso

✅ **Troca de chaves**  
✅ **Assinatura digital**  
✅ **Certificados SSL/TLS**

---

**Status:** ✅ Completo
