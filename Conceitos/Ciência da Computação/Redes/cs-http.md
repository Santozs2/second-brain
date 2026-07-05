---
type: concept
area: Conceitos
difficulty: beginner
id: cs-http
category: Networking
tags:
  - networking
created: 2026-07-05
updated: 2026-07-05
---
# 🌐 HTTP

> HyperText Transfer Protocol - Protocolo da Web.

---

## 📊 Versões

```
HTTP/1.1 (1997)
  ├── Conexões persistentes
  ├── Pipeline (ainda lento)
  └── Latência problema

HTTP/2 (2015)
  ├── Multiplexing (múltiplas streams)
  ├── Server push
  ├── Compressão headers (HPACK)
  └── 50% mais rápido

HTTP/3 (2022)
  ├── QUIC (UDP-based)
  ├── Faster connection
  └── Melhor móvel (roaming)
```

---

## 💻 Métodos

```
GET     - Retrieve resource
POST    - Create resource
PUT     - Update full resource
PATCH   - Update partial
DELETE  - Remove resource
HEAD    - GET sem body
OPTIONS - Available methods
```

---

## 📊 Status Codes

```
1xx - Informational
2xx - Success (200 OK, 201 Created)
3xx - Redirect (301, 302, 304)
4xx - Client error (400, 401, 404)
5xx - Server error (500, 502, 503)
```

---

## 🎯 Caching

```
Cache-Control: max-age=3600
ETag: "hash"
Etag usado para validação
```

---

## 🔗 Relacionado

- [[cs-ssl-tls|HTTPS/TLS]]
- [[cs-tcp|TCP]]

---

**Status:** ✅ Completo
