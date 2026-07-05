---
type: tech
area: Estudos
id: fw-nuxt-ssr
category: Nuxt
created: 2026-07-05
updated: 2026-07-05
---
# 🎯 Nuxt - Server-Side Rendering

---

## 🚀 SSR Out-of-Box

```javascript
// nuxt.config.ts
export default defineNuxtConfig({
  ssr: true  // Enabled by default
});
```

---

## 📦 useAsyncData

```vue
<script setup>
const { data: posts } = await useFetch('/api/posts');
</script>

<template>
  <div v-for="post in posts" :key="post.id">
    {{ post.title }}
  </div>
</template>
```

---

## 💾 Payload Hydration

Server sends data to client, no refetch needed.

---

## 🌳 Layouts

```vue
<!-- app.vue -->
<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>
```

---

**Status:** ✅ Completo
