---
type: technology
id: fw-svelte-overview
created: 2026-07-05
category: Svelte
---

# 🔥 Svelte - Complete Guide

---

## 📊 Overview

**Type:** Compiler, not framework  
**Paradigm:** Reactive, compiler-based  
**Bundle:** Small (~3KB)  
**Creator:** Rich Harris  

---

## 🎯 Philosophy

No runtime overhead. Compile to vanilla JS.

---

## 🚀 Features

- **Reactive assignments**
- **Scoped styles**
- **Animations**
- **Transitions**
- **Two-way binding**
- **Stores** (state management)

---

## 💻 Example

```svelte
<script>
  let count = 0;
  function increment() {
    count++;
  }
</script>

<button on:click={increment}>
  Count: {count}
</button>

<style>
  button { color: blue; }
</style>
```

---

**Status:** ✅ Completo
