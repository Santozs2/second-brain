---
type: technology
id: fw-react-performance
created: 2026-07-05
category: React
---

# ⚛️ React - Performance Optimization

---

## 🚀 Code Splitting

```javascript
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./Heavy'));

export function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}
```

---

## 💾 Memoization

```javascript
import { memo, useMemo, useCallback } from 'react';

const OptimizedChild = memo(({ items }) => (
  <div>{items.map(i => <div key={i.id}>{i.name}</div>)}</div>
));

function Parent() {
  const items = useMemo(() => expensiveComputation(), []);
  const handleClick = useCallback(() => {}, []);
  
  return <OptimizedChild items={items} onClick={handleClick} />;
}
```

---

## 📊 Metrics

- **First Contentful Paint (FCP):** < 1.8s
- **Largest Contentful Paint (LCP):** < 2.5s
- **Cumulative Layout Shift (CLS):** < 0.1
- **Time to Interactive (TTI):** < 3.8s

---

## 🎯 Best Practices

1. Use React DevTools Profiler
2. Code split large applications
3. Lazy load images
4. Minimize bundle size
5. Use production build
6. Monitor Core Web Vitals

---

**Status:** ✅ Completo
