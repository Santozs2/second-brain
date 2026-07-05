---
type: concept
area: Conceitos
difficulty: beginner
id: cs-dns
category: Networking
tags:
  - networking
created: 2026-07-05
updated: 2026-07-05
---
# 🌐 DNS (Domain Name System)

> Traduzir nomes de domínio → endereços IP.

---

## 📊 Hierarquia

```
example.com
├── Root nameserver    (.)
├── TLD nameserver     (.com)
├── Authoritative NS   (example.com)
└── Local resolver     (seu ISP)
```

---

## 🔄 Resolução

```
1. Client → Resolver: "example.com?"
2. Resolver → Root: "Quem sabe .com?"
3. Root → TLD: "Pergunte a example.com NS"
4. Resolver → Auth NS: "example.com?"
5. Auth NS → "93.184.216.34"
6. Resolver → Client: "93.184.216.34"
```

---

## 💾 Caching

DNS resposta é cacheada (TTL)  
TTL baixo = updates rápidos, mais queries  
TTL alto = menos queries, updates lentos

---

## 🔒 DNSSEC

Assinatura digital para evitar spoofing

---

## 🎯 Tipos de registro

```
A     → IPv4
AAAA  → IPv6
CNAME → Alias
MX    → Mail server
TXT   → Texto (SPF, DKIM)
```

---

**Status:** ✅ Completo
