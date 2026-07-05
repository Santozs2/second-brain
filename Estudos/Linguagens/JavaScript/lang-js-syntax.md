---
type: technology
id: lang-js-syntax
created: 2026-07-05
updated: 2026-07-05
category: JavaScript
tags:
  - type/technology
  - domain/frontend
  - language/javascript
---

# 🟨 JavaScript - Syntax & Types

---

## 📖 Tipos Primitivos

```javascript
// Primitivos
typeof 42            // "number"
typeof "hello"       // "string"
typeof true          // "boolean"
typeof undefined     // "undefined"
typeof Symbol("x")   // "symbol"
typeof 42n           // "bigint"

// Objetos
typeof {}            // "object"
typeof []            // "object" (array é object!)
typeof null          // "object" (bug histórico)
typeof function(){}  // "function"
```

---

## 📦 Variáveis

```javascript
var x = 1;        // Function scope, hoisted
let y = 2;        // Block scope, não hoisted
const z = 3;      // Block scope, não reassign

// Hoisting
console.log(x);   // undefined (declaração hoisted)
var x = 1;
```

---

## 🔄 Operadores

```javascript
// Comparação
5 == "5"    // true (coerção de tipo)
5 === "5"   // false (tipo igual)

// Lógicos
true && false   // false
true || false   // true
!true          // false

// Nullish coalescing
null ?? 'default'    // 'default'

// Optional chaining
obj?.prop?.method?.()
```

---

## 🎯 Control Flow

```javascript
if (x > 5) {
} else if (x > 2) {
} else {
}

switch (x) {
  case 1:
    break;
  default:
}

for (let i = 0; i < 10; i++) {}
while (true) {}
do {} while (false);
```

---

**Status:** ✅ Completo
