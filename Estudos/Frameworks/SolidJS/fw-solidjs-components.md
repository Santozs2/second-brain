---
type: tech
area: Estudos
id: fw-solidjs-components
category: SolidJS
created: 2026-07-05
updated: 2026-07-05
---
# 🟦 SolidJS - Components

---

## 📝 Function Components

```javascript
function Button(props) {
  return <button onclick={props.onclick}>{props.label}</button>;
}
```

---

## 🔗 Props

```javascript
function Card(props) {
  return <div>{props.children}</div>;
}
```

---

## 📊 Show/For

```javascript
<Show when={isVisible()}>
  <div>Visible</div>
</Show>

<For each={items()}>
  {(item) => <div>{item}</div>}
</For>
```

---

**Status:** ✅ Completo
