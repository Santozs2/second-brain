---
type: tech
area: Estudos
id: fw-vue-components
category: Vue
created: 2026-07-05
updated: 2026-07-05
---
# 💚 Vue - Components

---

## 📝 Single-File Component

```vue
<template>
  <div class="card">
    <h2>{{ title }}</h2>
    <p>{{ content }}</p>
    <button @click="count++">Count: {{ count }}</button>
  </div>
</template>

<script setup>
import { ref } from 'vue';
const count = ref(0);
const title = ref('Hello');
const content = ref('Vue component');
</script>

<style scoped>
.card { padding: 20px; }
</style>
```

---

## 🔄 Props & Emits

```vue
<script setup>
defineProps(['title', 'items']);
defineEmits(['update', 'delete']);
</script>

<template>
  <div @click="$emit('update', newValue)">
    {{ title }}: {{ items.length }}
  </div>
</template>
```

---

## 📦 Slots

```vue
<template>
  <div>
    <slot name="header">Default Header</slot>
    <slot>Default content</slot>
  </div>
</template>
```

---

**Status:** ✅ Completo
