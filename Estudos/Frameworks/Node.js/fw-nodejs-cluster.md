---
type: tech
area: Estudos
id: fw-nodejs-cluster
category: Node.js
created: 2026-07-05
updated: 2026-07-05
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
