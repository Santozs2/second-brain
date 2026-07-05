---
type: concept
id: cs-ssl-tls
created: 2026-07-05
updated: 2026-07-05
category: Networking
tags:
  - type/concept
  - domain/networking
  - difficulty/advanced
---

# 🔐 SSL/TLS

> Protocolo de segurança para web.

---

## 🤝 TLS Handshake

```
1. ClientHello (cipher suites, versão)
2. ServerHello (escolhe cipher suite)
3. ServerCertificate (publica chave)
4. ServerKeyExchange (parâmetros)
5. ServerHelloDone
6. ClientKeyExchange (session key encriptada)
7. Finish (MAC de tudo)
```

---

## 📊 Versões

```
SSL 2.0 - Quebrado, inseguro
SSL 3.0 - Quebrado
TLS 1.0 - Antigo
TLS 1.2 - Padrão atual
TLS 1.3 - Novo (mais rápido, mais seguro)
```

---

## 🔒 Certificados

Assinado por CA (Certification Authority)  
Validação de identidade do servidor

---

**Status:** ✅ Completo
