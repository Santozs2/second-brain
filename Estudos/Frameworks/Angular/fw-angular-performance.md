---
type: tech
area: Estudos
id: fw-angular-performance
category: Angular
created: 2026-07-05
updated: 2026-07-05
---
# 🅰️ Angular - Performance

---

## 🚀 Optimization

- **OnPush strategy** - Manual change detection
- **Lazy loading** - Load modules on demand
- **Tree shaking** - Remove unused code
- **AOT compilation** - Ahead-of-time

---

## 📦 Lazy Loading

```typescript
const routes = [
  { path: 'lazy', loadChildren: () => import('./lazy/lazy.module').then(m => m.LazyModule) }
];
```

---

## 💾 Change Detection

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
```

---

## 📊 Metrics

Bundle size, load time, TTI.

---

**Status:** ✅ Completo
