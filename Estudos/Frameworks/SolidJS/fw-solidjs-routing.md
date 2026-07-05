---
type: tech
area: Estudos
id: fw-solidjs-routing
category: SolidJS
created: 2026-07-05
updated: 2026-07-05
---
# 🟦 SolidJS - Routing

---

## 🛣️ Solid Router

```javascript
import { Router, Route } from '@solidjs/router';

function App() {
  return (
    <Router>
      <Route path="/" component={Home} />
      <Route path="/about" component={About} />
    </Router>
  );
}
```

---

## 🔗 Navigation

```javascript
import { useNavigate } from '@solidjs/router';

function Nav() {
  const navigate = useNavigate();
  return <button onclick={() => navigate('/')}>Home</button>;
}
```

---

**Status:** ✅ Completo
