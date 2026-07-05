---
type: technology
id: fw-nextjs-deploy
created: 2026-07-05
category: Next.js
---

# ⚡ Next.js - Deployment

---

## 🚀 Vercel (Recommended)

```bash
npm install -g vercel
vercel login
vercel
```

Auto-deploys on push. Best for Next.js.

---

## 🐳 Docker

```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY .next ./.next
EXPOSE 3000
CMD ["npm", "start"]
```

---

## ☁️ Self-Hosted

```bash
npm run build
npm start
```

---

## 📊 Environment Variables

```bash
# .env.local
DATABASE_URL=postgresql://...
API_KEY=secret
```

---

## 🎯 Optimization

- Bundle analysis
- Image optimization
- Code splitting
- Caching strategies

---

**Status:** ✅ Completo
