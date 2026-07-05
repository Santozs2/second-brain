---
type: technology
id: fw-nodejs-error-handling
created: 2026-07-05
category: Node.js
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
