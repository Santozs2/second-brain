---
type: technology
id: fw-nodejs-async
created: 2026-07-05
category: Node.js
---

# 🟢 Node.js - Async/Promises

---

## 🔄 Callbacks

```javascript
fs.readFile('file.txt', (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

---

## 💬 Promises

```javascript
fs.promises.readFile('file.txt')
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

---

## 🎯 Async/Await

```javascript
async function read() {
  try {
    const data = await fs.promises.readFile('file.txt');
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}
```

---

**Status:** ✅ Completo
