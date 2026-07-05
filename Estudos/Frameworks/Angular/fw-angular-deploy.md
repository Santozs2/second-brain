---
type: tech
area: Estudos
id: fw-angular-deploy
category: Angular
created: 2026-07-05
updated: 2026-07-05
---
# 🅰️ Angular - Deployment

---

## 🏗️ Build

```bash
ng build --prod
```

---

## ☁️ Hosting

- Firebase Hosting
- Vercel
- Netlify
- AWS S3

---

## 🐳 Docker

```dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install && npm run build
EXPOSE 4200
CMD ["npm", "start"]
```

---

**Status:** ✅ Completo
