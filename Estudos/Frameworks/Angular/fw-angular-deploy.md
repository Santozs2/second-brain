---
type: technology
id: fw-angular-deploy
created: 2026-07-05
category: Angular
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
