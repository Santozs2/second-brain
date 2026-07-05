---
type: tech
area: Estudos
id: fw-express-auth
category: Express
created: 2026-07-05
updated: 2026-07-05
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
