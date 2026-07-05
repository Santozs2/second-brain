---
type: concept
id: cs-encryption
created: 2026-07-05
updated: 2026-07-05
category: Security
tags:
  - type/concept
  - domain/security
  - difficulty/intermediate
---

# 🔐 Encryption

> Transformar dados legíveis em ilegíveis sem chave.

---

## 🔑 Tipos

### Simétrica (mesma chave)
```
Plaintext + Key → Cipher + Key → Plaintext
Rápida (1000x mais que assimétrica)
Problema: Distribuir chave
```

**Exemplos:** AES-256, DES (inseguro)

### Assimétrica (chaves diferentes)
```
Plaintext + Public Key → Cipher
Cipher + Private Key → Plaintext
Lenta, mas não precisa compartilhar private key
```

**Exemplos:** RSA, ECDSA

### Híbrida (ambas)
```
1. Assimétrica para trocar chave simétrica
2. Simétrica para criptografar dados
Melhor dos dois mundos
```

---

## 🎯 Modos

**ECB (inseguro):** Blocks iguais → ciphers iguais  
**CBC:** Cada block depende do anterior  
**GCM:** Autenticação + criptografia

---

## 🔒 IND-CPA

Criptografia é segura se adversário não consegue  
distinguir entre encrypção de 2 mensagens

---

**Status:** ✅ Completo
