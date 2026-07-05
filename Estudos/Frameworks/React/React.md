---
type: tech
tags:
  - tech
  - estudo
  - frontend
tecnologia: React
status: aprendendo
created: 2026-06-30
updated: 2026-06-30
---

# React

> [!tip] Status
> 🟡 Aprendendo

## 📝 Resumo

React é uma biblioteca para construir interfaces de usuário baseadas em componentes reutilizáveis e estado reativo.

## 🧠 Conceitos principais

- **Componentes**: funcionais, props
- **Hooks**: `useState`, `useEffect`, `useContext`, `useRef`
- **Renderização condicional e listas** (`key`)
- **Eventos e formulários controlados**
- **Context API** para estado global simples
- **Custom hooks**

## 💻 Exemplos

```tsx
function Contador() {
  const [contagem, setContagem] = useState(0);

  return (
    <button onClick={() => setContagem(contagem + 1)}>
      Cliques: {contagem}
    </button>
  );
}
```

## 🔗 Links úteis

- [Documentação oficial React](https://react.dev/)
- [React - Aprenda](https://react.dev/learn)

## ✅ Checklist de aprendizado

- [x] Componentes e props
- [ ] `useState` e `useEffect`
- [ ] Formulários controlados
- [ ] Context API
- [ ] Custom hooks

## 🗒️ Notas pessoais


## 🔗 Veja também

- [[JavaScript|JavaScript]]
- [[TypeScript|TypeScript]]
- [[Next.js|Next.js]]
