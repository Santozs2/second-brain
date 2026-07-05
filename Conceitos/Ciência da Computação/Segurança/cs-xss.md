---
type: concept
id: cs-xss
created: 2026-07-05
updated: 2026-07-05
category: Security
tags:
  - type/concept
  - domain/security
  - difficulty/beginner
---

# ⚠️ XSS (Cross-Site Scripting)

> Injetar JavaScript malicioso em página.

---

## 🎯 Tipos

### Stored XSS
```
1. Atacante submete <script>alert('hack')</script>
2. Server armazena sem validar
3. Vítima lê post → script executa
4. Roubar cookies, mudar página, etc
```

### Reflected XSS
```
URL: /search?q=<script>alert('hack')</script>
Server reflecte sem escapar
Script executa no navegador da vítima
```

---

## 🛡️ Proteção

### Input Validation
```python
if '<script>' in user_input:
    return "Inválido!"
```

### Output Encoding
```python
# Escapar caracteres especiais
html.escape(user_input)  # < > & → &lt; &gt; &amp;
```

### Content Security Policy (CSP)
```
Header: Content-Security-Policy: script-src 'self'
Bloqueia scripts de fora
```

### HTTPOnly Cookies
```
Set-Cookie: session=abc; HttpOnly
JavaScript não consegue acessar
```

---

## 🎯 Prevenção

✅ Nunca confie em input do usuário  
✅ Sempre escape output  
✅ Use templates seguros  
✅ CSP headers  
✅ HTTPOnly + Secure flags

---

**Status:** ✅ Completo
