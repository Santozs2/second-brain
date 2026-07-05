---
type: technology
id: fw-react-deploy
created: 2026-07-05
category: React
---

# ⚛️ React - Deployment

---

## 🐳 Docker

```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

---

## ☁️ Hosting Options

**Vercel** - Optimized for React
**Netlify** - Static + serverless
**AWS Amplify** - Full AWS stack
**GitHub Pages** - Free static
**Railway/Render** - Simple deployment

---

## 🚀 Build Optimization

```bash
npm run build
npm run analyze  # Bundle analysis
```

---

## 📊 CI/CD

```yaml
# GitHub Actions
name: Deploy
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: npm install && npm run build
      - run: npm run test
```

---

**Status:** ✅ Completo
