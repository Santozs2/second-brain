---
type: tech
area: Estudos
id: fw-nuxt-auto-routing
category: Nuxt
created: 2026-07-05
updated: 2026-07-05
---
# 🎯 Nuxt - Auto Routing

---

## 📂 File-Based Routing

File structure automatically creates routes.

```
pages/
  index.vue           → /
  about.vue           → /about
  products/
    index.vue         → /products
    [id].vue          → /products/:id
  user/
    profile.vue       → /user/profile
```

---

## 🔗 Navigation

```vue
<NuxtLink to="/about">About</NuxtLink>
<NuxtPage />
```

---

## 🎯 Dynamic Routes

```vue
<!-- pages/posts/[id].vue -->
<script setup>
const { id } = useRoute().params;
</script>
```

---

## 🌳 Layout System

Wrap pages with layouts automatically.

---

**Status:** ✅ Completo
