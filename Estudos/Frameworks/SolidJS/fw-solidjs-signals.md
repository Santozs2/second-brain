---
type: technology
id: fw-solidjs-signals
created: 2026-07-05
category: SolidJS
---

# 🟦 SolidJS - Signals

---

## 📡 Create Signals

```javascript
const [count, setCount] = createSignal(0);
const [user, setUser] = createSignal(null);
```

---

## 📊 Computed

```javascript
const doubled = createMemo(() => count() * 2);
```

---

## 👀 Effects

```javascript
createEffect(() => {
  console.log('Count changed:', count());
});
```

---

## 🔄 Updates

```javascript
setCount(c => c + 1);
```

---

**Status:** ✅ Completo
