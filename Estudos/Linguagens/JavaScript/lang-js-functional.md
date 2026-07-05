---
type: tech
area: Estudos
id: lang-js-functional
category: JavaScript
tags:
  - javascript
created: 2026-07-05
updated: 2026-07-05
---
# 🟨 JavaScript - Functional Programming

---

## 🔧 Pure Functions

```javascript
// Pure
const add = (a, b) => a + b;

// Impure
let total = 0;
const addToTotal = (x) => total += x;
```

---

## 🔄 Closures

```javascript
function outer(x) {
  return function inner(y) {
    return x + y;
  };
}

const add5 = outer(5);
add5(3); // 8
```

---

## 📊 Higher-Order Functions

```javascript
const map = (fn, arr) => arr.map(fn);
const filter = (fn, arr) => arr.filter(fn);
const compose = (f, g) => (x) => f(g(x));

const numbers = [1, 2, 3, 4];
map(x => x * 2, numbers); // [2,4,6,8]
```

---

**Status:** ✅ Completo
