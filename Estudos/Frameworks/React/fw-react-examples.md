---
type: tech
area: Estudos
id: fw-react-examples
category: React
created: 2026-07-05
updated: 2026-07-05
---
# ⚛️ React - Code Examples & Projects

---

## 📱 Todo App

```javascript
import { useState } from 'react';

export default function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');

  const addTodo = () => {
    setTodos([...todos, { id: Date.now(), text: input }]);
    setInput('');
  };

  return (
    <div>
      <input value={input} onChange={e => setInput(e.target.value)} />
      <button onClick={addTodo}>Add</button>
      <ul>
        {todos.map(t => <li key={t.id}>{t.text}</li>)}
      </ul>
    </div>
  );
}
```

---

## 🔄 API Integration

```javascript
import { useEffect, useState } from 'react';

export function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/users')
      .then(r => r.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []);

  if (loading) return <div>Loading...</div>;
  return <div>{users.map(u => <div key={u.id}>{u.name}</div>)}</div>;
}
```

---

## 🎨 Real Projects

- E-commerce app
- Social media
- Dashboard
- Chat application
- Blog platform

---

**Status:** ✅ Completo
