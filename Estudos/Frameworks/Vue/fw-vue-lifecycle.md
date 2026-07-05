---
type: tech
area: Estudos
id: fw-vue-lifecycle
category: Vue
created: 2026-07-05
updated: 2026-07-05
---
# 💚 Vue - Lifecycle

---

## 📊 Vue 3 Composition API

```javascript
import { onMounted, onUnmounted } from 'vue';

export default {
  setup() {
    onMounted(() => console.log('mounted'));
    onUnmounted(() => console.log('unmounted'));
    
    return {};
  }
};
```

---

## 🔄 Hooks

- **onBeforeMount** - Before render
- **onMounted** - After render
- **onBeforeUpdate** - Before data change
- **onUpdated** - After data change
- **onBeforeUnmount** - Before destroy
- **onUnmounted** - After destroy

---

## 💾 Cleanup

```javascript
onMounted(() => {
  const listener = () => {};
  window.addEventListener('resize', listener);
  
  onUnmounted(() => {
    window.removeEventListener('resize', listener);
  });
});
```

---

**Status:** ✅ Completo
