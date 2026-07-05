---
type: tech
area: Estudos
id: fw-express-overview
category: Express
created: 2026-07-05
updated: 2026-07-05
---
# 🚂 Express - Minimal Web Framework

---

## 📊 Overview

**Type:** Minimal web framework for Node.js  
**Use:** REST APIs, web servers  
**Popularity:** Most used backend framework  

---

## 💻 Example

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.json({ message: 'Hello' });
});

app.listen(3000);
```

---

## 🎯 Features

- Routing
- Middleware
- Static files
- Error handling

---

**Status:** ✅ Completo

## 🗂️ Neste guia

- [[fw-express-auth]]
- [[fw-express-database]]
- [[fw-express-error]]
- [[fw-express-middleware]]
- [[fw-express-routing]]
- [[fw-express-testing]]
