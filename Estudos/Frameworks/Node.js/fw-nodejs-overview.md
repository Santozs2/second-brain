---
type: tech
area: Estudos
id: fw-nodejs-overview
category: Node.js
created: 2026-07-05
updated: 2026-07-05
---
# 🟢 Node.js - Complete Guide

---

## 📊 Overview

**Type:** JavaScript runtime  
**Engine:** V8  
**Use:** Backend, CLI, tools  
**Market:** 50% of developers  

---

## 🚀 Features

- **Non-blocking I/O**
- **Event-driven**
- **NPM ecosystem** (2.5M packages)
- **Single-threaded** (async)
- **Module system**

---

## 💻 Example

```javascript
const http = require('http');
const server = http.createServer((req, res) => {
  res.writeHead(200);
  res.end('Hello World');
});
server.listen(3000);
```

---

## 📦 Package Manager

NPM - Node Package Manager

---

**Status:** ✅ Completo

## 🗂️ Neste guia

- [[fw-nodejs-async]]
- [[fw-nodejs-cluster]]
- [[fw-nodejs-deploy]]
- [[fw-nodejs-error-handling]]
- [[fw-nodejs-modules]]
- [[fw-nodejs-streams]]
- [[fw-nodejs-testing]]
