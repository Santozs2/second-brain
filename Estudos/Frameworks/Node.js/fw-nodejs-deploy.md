---
type: tech
area: Estudos
id: fw-nodejs-deploy
category: Node.js
created: 2026-07-05
updated: 2026-07-05
---
# 🟢 Node.js - Deployment

---

## 🐳 Docker

```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
CMD ["npm", "start"]
```

---

## ☁️ Platforms

Heroku, Railway, Vercel, AWS Lambda.

---

**Status:** ✅ Completo
