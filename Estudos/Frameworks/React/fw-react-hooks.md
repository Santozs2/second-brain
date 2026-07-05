---
type: technology
id: fw-react-hooks
created: 2026-07-05
category: React
---

# ⚛️ React - Hooks Deep Dive

---

## 📊 Essential Hooks

### useState
```javascript
const [state, setState] = useState(initialValue);
setState(newValue);
setState(prev => prev + 1); // Functional update
```

### useEffect
Manage side effects (API calls, subscriptions)

### useContext
Access context values without prop drilling

### useReducer
Complex state logic with reducer function

### useCallback
Memoize function references (prevent re-renders)

### useMemo
Memoize expensive computations

### useRef
Access DOM directly or store mutable value

---

## 🎯 Custom Hooks

```javascript
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);
  
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return width;
}

// Usage
function Component() {
  const width = useWindowWidth();
  return <div>{width}px</div>;
}
```

---

## ⚠️ Rules of Hooks

1. Only call at top level
2. Only in React functions
3. Call same number each render

---

**Status:** ✅ Completo
