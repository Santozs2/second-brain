---
type: concept
id: cs-sql-injection
created: 2026-07-05
updated: 2026-07-05
category: Security
tags:
  - type/concept
  - domain/security
  - difficulty/beginner
---

# 💉 SQL Injection

> Atacante injeta código SQL malicioso.

---

## 🚨 Exemplo

```python
# INSEGURO
query = "SELECT * FROM users WHERE username = '" + username + "'"

# Atacante entra: ' OR '1'='1
# Query fica: SELECT * FROM users WHERE username = '' OR '1'='1'
# Retorna TODOS usuários!
```

---

## 🛡️ Proteção

### 1. Prepared Statements (MELHOR)
```python
# SEGURO
cursor.execute("SELECT * FROM users WHERE username = ?", (username,))
```

### 2. Input Validation
```python
# Valida tipo e tamanho
if len(username) > 50:
    return error
```

### 3. Escaping
```python
# Escapa caracteres especiais
username = username.replace("'", "''")
```

### 4. Least Privilege
```
DB user tem permissão mínima
Só SELECT, não DELETE/UPDATE
```

---

## 🎯 Impacto

✅ Roubar dados (credenciais)  
✅ Modificar dados  
✅ Deletar tudo  
✅ Burlar autenticação

---

## ✅ Checklist

- [ ] Use prepared statements
- [ ] Valide inputs
- [ ] Escape outputs
- [ ] Log queries suspeitas
- [ ] WAF (Web Application Firewall)

---

**Status:** ✅ Completo
