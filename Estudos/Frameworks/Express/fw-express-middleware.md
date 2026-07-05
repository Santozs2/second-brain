---
type: technology
id: fw-express-middleware
created: 2026-07-05
category: Express
---

# 🚂 Express - Middleware
```javascript
app.use(express.json());
app.use((req, res, next) => { console.log(req.method); next(); });
app.use('/api', authMiddleware);
```
**Status:** ✅ Completo
