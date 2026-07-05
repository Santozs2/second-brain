---
type: concept
area: Conceitos
difficulty: beginner
id: cs-csrf
category: Security
tags:
  - security
created: 2026-07-05
updated: 2026-07-05
---
# 🚨 CSRF (Cross-Site Request Forgery)

> Atacante faz vítima executar ação sem consentimento.

---

## 🎯 Ataque

```
1. Vítima logged em banco.com
2. Visitab malicioso.com
3. Malicioso contém: <img src="banco.com/transfer?to=hacker&amount=1000">
4. Browser envia cookie automaticamente
5. Transferência acontece!
```

---

## 🛡️ Proteção

### CSRF Token
```python
# Servidor gera token único por sessão
<form>
    <input type="hidden" name="csrf_token" value="abc123">
    <input type="submit">
</form>

# Server valida token antes de processar
if request.csrf_token != session.csrf_token:
    return "Inválido"
```

### SameSite Cookie
```
Set-Cookie: session=abc; SameSite=Strict
Navegador só envia em requests do mesmo site
```

### Double Submit Cookie
```
Token em cookie E em form
Server compara se são iguais
```

---

## vs XSS

XSS: Executa code no navegador  
CSRF: Usa cookies válidos do navegador

---

## ✅ Checklist

- [ ] CSRF tokens em forms
- [ ] SameSite cookies
- [ ] Validar origin/referer
- [ ] POST para ações críticas

---

**Status:** ✅ Completo
