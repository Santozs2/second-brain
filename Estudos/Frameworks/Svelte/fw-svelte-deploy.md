---
type: technology
id: fw-svelte-deploy
created: 2026-07-05
category: Svelte
---

# 🔥 Svelte - Deployment

---

## 🚀 Vercel

```bash
npm run build
vercel
```

---

## 🐳 Docker

```dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install && npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

---

**Status:** ✅ Completo
