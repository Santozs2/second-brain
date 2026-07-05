---
type: tech
area: Estudos
id: fw-nuxt-deploy
category: Nuxt
created: 2026-07-05
updated: 2026-07-05
---
# 🎯 Nuxt - Deployment

---

## 🚀 Vercel

Best for Nuxt. Auto-deployment on push.

```bash
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
CMD ["npm", "run", "preview"]
```

---

## 🌍 Static Hosting

Generate static site:
```bash
npm run generate
```

Deploy `dist/` folder.

---

**Status:** ✅ Completo
