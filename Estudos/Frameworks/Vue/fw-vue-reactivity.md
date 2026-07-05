---
type: technology
id: fw-vue-reactivity
created: 2026-07-05
category: Vue
---

# 💚 Vue - Reactivity

---

## 📊 Ref & Reactive

```javascript
import { ref, reactive } from 'vue';

// Ref for primitives
const count = ref(0);
count.value++;

// Reactive for objects
const state = reactive({ user: { name: 'John' } });
state.user.name = 'Jane';
```

---

## 🔄 Computed

```javascript
const doubled = computed(() => count.value * 2);
```

---

## 👀 Watch

```javascript
watch(count, (newVal, oldVal) => {
  console.log(`Count changed from ${oldVal} to ${newVal}`);
});
```

---

## ⚡ Watchers

Computed vs Watch - use computed for derived state.

---

**Status:** ✅ Completo
