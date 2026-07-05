---
type: technology
id: fw-react-lifecycle
created: 2026-07-05
category: React
---

# ⚛️ React - Component Lifecycle

---

## 📊 Functional Components with Hooks

Modern React uses hooks instead of class lifecycle methods.

---

## 🔄 Hook Lifecycle

```javascript
import { useEffect, useState } from 'react';

export function Component() {
  const [count, setCount] = useState(0);

  // Runs after render (equivalent to componentDidMount + componentDidUpdate)
  useEffect(() => {
    console.log('Component mounted or updated');
    
    // Cleanup function (equivalent to componentWillUnmount)
    return () => {
      console.log('Component unmounting or dependency changed');
    };
  }, [count]); // Dependency array

  return <div>{count}</div>;
}
```

---

## 📋 Common Patterns

```javascript
// Mount only
useEffect(() => {
  // fetch data
}, []);

// Updates
useEffect(() => {
  // runs on every render
});

// Specific dependency
useEffect(() => {
  // runs when dep changes
}, [dependency]);

// Custom hook
function useCustom() {
  const [state, setState] = useState(null);
  useEffect(() => {
    // logic
  }, []);
  return state;
}
```

---

## 🎯 Best Practices

- Keep effects focused and single-purpose
- Use dependency arrays correctly
- Cleanup side effects properly
- Avoid infinite loops

---

**Status:** ✅ Completo
