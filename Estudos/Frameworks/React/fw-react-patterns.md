---
type: technology
id: fw-react-patterns
created: 2026-07-05
category: React
---

# ⚛️ React - Design Patterns

---

## 🏗️ Component Composition

```javascript
// Composition over Inheritance
function Card({ children, title }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      {children}
    </div>
  );
}

export function Page() {
  return (
    <Card title="Welcome">
      <p>Content here</p>
    </Card>
  );
}
```

---

## 🎯 Render Props

```javascript
function Mouse({ children }) {
  const [pos, setPos] = useState({ x: 0, y: 0 });
  
  return (
    <div onMouseMove={(e) => setPos({ x: e.x, y: e.y })}>
      {children(pos)}
    </div>
  );
}

<Mouse>
  {({ x, y }) => <div>Mouse: {x}, {y}</div>}
</Mouse>
```

---

## 🔗 Higher-Order Components

```javascript
function withTheme(Component) {
  return function WithTheme(props) {
    const [theme, setTheme] = useState('light');
    return <Component {...props} theme={theme} />;
  };
}

export default withTheme(MyComponent);
```

---

## 🎪 Container/Presentational

```javascript
// Container (logic)
function UserContainer() {
  const [user, setUser] = useState(null);
  useEffect(() => { fetchUser(); }, []);
  return <UserDisplay user={user} />;
}

// Presentational (UI)
function UserDisplay({ user }) {
  return <div>{user?.name}</div>;
}
```

---

**Status:** ✅ Completo
