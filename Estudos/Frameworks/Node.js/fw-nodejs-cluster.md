---
type: technology
id: fw-nodejs-cluster
created: 2026-07-05
category: Node.js
---

# 🟢 Node.js - Clustering

---

## 🔄 Multiple Processes

```javascript
const cluster = require('cluster');
if (cluster.isMaster) {
  for (let i = 0; i < 4; i++) cluster.fork();
} else {
  http.createServer().listen(3000);
}
```

---

## ⚡ Performance

Utilize all CPU cores.

---

**Status:** ✅ Completo
