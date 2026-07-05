---
type: tech
area: Estudos
id: fw-svelte-overview
category: Svelte
created: 2026-07-05
updated: 2026-07-05
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

## 🗂️ Neste guia

- [[fw-svelte-animations]]
- [[fw-svelte-deploy]]
- [[fw-svelte-events]]
- [[fw-svelte-reactive]]
- [[fw-svelte-sveltekit]]
- [[fw-svelte-testing]]
