---
type: tech
area: Estudos
id: fw-express-db
category: Express
created: 2026-07-05
updated: 2026-07-05
---
# 🚂 Express - Database
```javascript
const mongoose = require('mongoose');
mongoose.connect('mongodb://');
const schema = new mongoose.Schema({ name: String });
const User = mongoose.model('User', schema);
```
**Status:** ✅ Completo
