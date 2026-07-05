---
type: technology
id: lang-js-async
created: 2026-07-05
updated: 2026-07-05
category: JavaScript
tags:
  - type/technology
  - language/javascript
---

# 🟨 JavaScript - Async Programming

---

## 🔄 Callbacks

```javascript
function fetchData(callback) {
  setTimeout(() => {
    callback("Data loaded");
  }, 1000);
}

fetchData((data) => console.log(data));
```

---

## 💬 Promises

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve("Success"), 1000);
});

promise
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

---

## 🎯 Async/Await

```javascript
async function getData() {
  try {
    const data = await fetch('/api/data');
    const json = await data.json();
    return json;
  } catch (err) {
    console.error(err);
  }
}

getData().then(data => console.log(data));
```

---

## 📊 Event Loop

- Call stack
- Microtask queue (promises)
- Macrotask queue (setTimeout)

---

**Status:** ✅ Completo
