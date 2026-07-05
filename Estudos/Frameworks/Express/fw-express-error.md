---
type: technology
id: fw-express-error
created: 2026-07-05
category: Express
---

# 🚂 Express - Error Handling
```javascript
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message });
});
```
**Status:** ✅ Completo
