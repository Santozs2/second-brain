---
type: technology
id: fw-nuxt-composables
created: 2026-07-05
category: Nuxt
---

# 🎯 Nuxt - Composables

---

## 🔧 Custom Composables

```javascript
// composables/useCount.ts
export const useCount = () => {
  const count = ref(0);
  const increment = () => count.value++;
  return { count, increment };
};
```

---

## 📱 Usage

```vue
<script setup>
const { count, increment } = useCount();
</script>

<template>
  <button @click="increment">{{ count }}</button>
</template>
```

---

## 🌐 Built-in Composables

- `useRouter()` - Router instance
- `useRoute()` - Current route
- `useFetch()` - Data fetching
- `useState()` - Shared state

---

**Status:** ✅ Completo
