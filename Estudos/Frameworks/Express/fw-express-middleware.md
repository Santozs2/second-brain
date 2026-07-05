---
type: tech
area: Estudos
id: fw-express-middleware
category: Express
created: 2026-07-05
updated: 2026-07-05
---
# 🚂 Express - Middleware
```javascript
app.use(express.json());
app.use((req, res, next) => { console.log(req.method); next(); });
app.use('/api', authMiddleware);
```
**Status:** ✅ Completo
