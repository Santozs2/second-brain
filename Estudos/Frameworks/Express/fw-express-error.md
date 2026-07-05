---
type: tech
area: Estudos
id: fw-express-error
category: Express
created: 2026-07-05
updated: 2026-07-05
---
# 🚂 Express - Error Handling
```javascript
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message });
});
```
**Status:** ✅ Completo
