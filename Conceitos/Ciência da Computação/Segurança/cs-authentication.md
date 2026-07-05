---
type: concept
id: cs-authentication
created: 2026-07-05
updated: 2026-07-05
category: Security
tags:
  - type/concept
  - domain/security
  - difficulty/beginner
---

# 🔐 Authentication

> Verificar identidade: "Você é quem diz ser?"

---

## 🔑 Fatores

### Algo que você SABE
```
Senha, PIN, security question
Fraco: Pode ser adivinhado/roubado
```

### Algo que você TEM
```
Token físico, smartphone, cartão
Mais seguro: Mais difícil robar
```

### Algo que você É
```
Biometria: Fingerprint, face, iris
Muito seguro: Não pode ser copiado
```

---

## 🔒 Senhas

```python
# INSEGURO
password_hash = md5(password)

# SEGURO
password_hash = bcrypt(password, rounds=12)
# Leva 100ms (resist brute force)
```

### Password Requirements
- 12+ caracteres
- Mix: Maiúscula, minúscula, número, símbolo
- Não palavras comuns
- Unique por site

---

## 🎯 MFA (Multi-Factor Authentication)

```
Usuário + Senha → Token SMS → Acesso
Quebra 1 fator = ainda seguro
```

**Tipos:**
- SMS (fraco, pode interceptar)
- App authenticator (TOTP - Time-based)
- Hardware key (U2F/FIDO2 - mais seguro)

---

## ✅ Best Practices

- [ ] Hash senhas (não store plaintext)
- [ ] MFA para contas importantes
- [ ] Senha recovery seguro
- [ ] Logout em todos devices
- [ ] Detectar tentativas suspeitas

---

**Status:** ✅ Completo
