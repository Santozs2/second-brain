---
type: technology
id: fw-nodejs-deploy
created: 2026-07-05
category: Node.js
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
