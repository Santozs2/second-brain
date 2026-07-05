---
type: technology
id: fw-express-auth
created: 2026-07-05
category: Express
---

# 🚂 Express - Authentication
```javascript
const jwt = require('jsonwebtoken');
app.post('/login', (req, res) => {
  const token = jwt.sign({ id: 1 }, 'secret');
  res.json({ token });
});
```
**Status:** ✅ Completo
