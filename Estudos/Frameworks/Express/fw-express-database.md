---
type: technology
id: fw-express-db
created: 2026-07-05
category: Express
---

# 🚂 Express - Database
```javascript
const mongoose = require('mongoose');
mongoose.connect('mongodb://');
const schema = new mongoose.Schema({ name: String });
const User = mongoose.model('User', schema);
```
**Status:** ✅ Completo
