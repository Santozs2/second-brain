---
type: tech
area: Estudos
id: fw-nextjs-security
category: Next.js
created: 2026-07-05
updated: 2026-07-05
---
# ⚡ Next.js - Security

---

## 🔐 CORS

```javascript
// pages/api/data.js
export default function handler(req, res) {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.status(200).json({ data: 'safe' });
}
```

---

## 🔒 Headers

```javascript
// next.config.js
const headers = async () => {
  return [{
    source: '/:path*',
    headers: [
      { key: 'X-Frame-Options', value: 'DENY' },
      { key: 'X-Content-Type-Options', value: 'nosniff' }
    ]
  }];
};
```

---

## 📝 Secrets

```bash
# .env.local
DATABASE_PASSWORD=secret
API_KEY=key
```

Never commit secrets.

---

## ✅ CSRF Protection

Use Next.js built-in CSRF protection.

---

**Status:** ✅ Completo
