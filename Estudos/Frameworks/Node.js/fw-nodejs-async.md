---
type: tech
area: Estudos
id: fw-nodejs-async
category: Node.js
created: 2026-07-05
updated: 2026-07-05
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
