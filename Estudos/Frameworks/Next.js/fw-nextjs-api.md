---
type: tech
area: Estudos
id: fw-nextjs-api
category: Next.js
created: 2026-07-05
updated: 2026-07-05
---
# ⚡ Next.js - API Routes

---

## 📝 Basic API

```javascript
// pages/api/hello.js
export default function handler(req, res) {
  if (req.method === 'POST') {
    const { name } = req.body;
    res.status(200).json({ msg: `Hello ${name}` });
  } else {
    res.status(200).json({ msg: 'Hello World' });
  }
}
```

---

## 🔐 Authentication

```javascript
export default function handler(req, res) {
  const token = req.headers.authorization;
  if (!token) return res.status(401).json({ error: 'Unauthorized' });
  
  // Verify token
  res.status(200).json({ data: 'secure' });
}
```

---

## 🗄️ Database

```javascript
import prisma from '@/lib/prisma';

export default async function handler(req, res) {
  const users = await prisma.user.findMany();
  res.json(users);
}
```

---

## ⚠️ Error Handling

```javascript
try {
  const data = await fetchData();
  res.status(200).json(data);
} catch (error) {
  res.status(500).json({ error: error.message });
}
```

---

**Status:** ✅ Completo
