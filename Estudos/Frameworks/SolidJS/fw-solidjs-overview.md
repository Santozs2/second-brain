---
type: tech
area: Estudos
id: fw-solidjs-overview
category: SolidJS
created: 2026-07-05
updated: 2026-07-05
---
# 🟦 SolidJS - Complete Guide

---

## 📊 Overview

**Type:** Reactive framework  
**Paradigm:** Fine-grained reactivity  
**Performance:** Fastest framework  
**Creator:** Ryan Carniato  

---

## 🎯 Philosophy

True reactivity. No virtual DOM.

---

## 💻 Example

```javascript
import { createSignal } from 'solid-js';

function Counter() {
  const [count, setCount] = createSignal(0);
  
  return (
    <button onClick={() => setCount(count() + 1)}>
      Count: {count()}
    </button>
  );
}
```

---

## 📦 Features

- **Signals** - Reactive values
- **Effects** - Side effects
- **Memo** - Computed values
- **JSX** - With fine-grained reactivity

---

**Status:** ✅ Completo
