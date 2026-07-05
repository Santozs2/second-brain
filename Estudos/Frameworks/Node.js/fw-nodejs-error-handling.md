---
type: tech
area: Estudos
id: fw-nodejs-error-handling
category: Node.js
created: 2026-07-05
updated: 2026-07-05
---
# 🟢 Node.js - Error Handling

---

## 🚨 Try/Catch

```javascript
try {
  riskyOperation();
} catch (err) {
  console.error(err);
}
```

---

## 🎯 Unhandled Promises

```javascript
process.on('unhandledRejection', (err) => {
  console.error(err);
  process.exit(1);
});
```

---

**Status:** ✅ Completo
