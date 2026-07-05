---
type: technology
id: fw-vue-router
created: 2026-07-05
category: Vue
---

# 💚 Vue - Vue Router

---

## 🛣️ Basic Setup

```javascript
import { createRouter, createWebHistory } from 'vue-router';

const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About },
  { path: '/user/:id', component: User }
];

const router = createRouter({
  history: createWebHistory(),
  routes
});

export default router;
```

---

## 🔗 Navigation

```vue
<router-link to="/about">About</router-link>
<router-view />
```

---

## 🎯 Params

```javascript
const id = route.params.id;
```

---

**Status:** ✅ Completo
