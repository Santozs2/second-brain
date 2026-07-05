---
type: tech
area: Estudos
id: fw-nuxt-middleware
category: Nuxt
created: 2026-07-05
updated: 2026-07-05
---
# 🎯 Nuxt - Middleware

---

## 🛡️ Route Middleware

```javascript
// middleware/auth.ts
export default defineRouteMiddleware((to, from) => {
  if (!isAuthenticated()) {
    return navigateTo('/login');
  }
});
```

---

## 🔐 Usage

```vue
<!-- pages/dashboard.vue -->
<script setup>
definePageMeta({
  middleware: 'auth'
});
</script>
```

---

## 🌍 Global Middleware

```javascript
// middleware/logger.global.ts
export default defineRouteMiddleware((to) => {
  console.log(`Navigating to ${to.path}`);
});
```

---

**Status:** ✅ Completo
