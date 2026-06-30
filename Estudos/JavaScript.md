---
type: tech
tags:
  - tech
  - estudo
  - frontend
  - backend
tecnologia: JavaScript
status: aprendendo
created: 2026-06-30
updated: 2026-06-30
---

# JavaScript

> [!tip] Status
> 🟡 Aprendendo

## 📝 Resumo

JavaScript é a linguagem de programação que dá interatividade à web. Roda no navegador e também no servidor (Node.js).

## 🧠 Conceitos principais

- **Tipos e variáveis**: `let`, `const`, tipos primitivos
- **Funções**: declaração, arrow functions, closures
- **Arrays e objetos**: métodos como `map`, `filter`, `reduce`
- **Assincronismo**: `Promise`, `async/await`, `fetch`
- **DOM**: seleção e manipulação de elementos, eventos
- **ES Modules**: `import`/`export`
- **Escopo e `this`**

## 💻 Exemplos

```js
async function getUsuarios() {
  const res = await fetch("https://api.exemplo.com/usuarios");
  const dados = await res.json();
  return dados.filter((u) => u.ativo);
}
```

## 🔗 Links úteis

- [MDN - JavaScript](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript)
- [JavaScript.info](https://javascript.info/)

## ✅ Checklist de aprendizado

- [x] Sintaxe básica e tipos
- [x] Funções e arrow functions
- [ ] Assincronismo (Promises/async-await)
- [ ] Manipulação de DOM e eventos
- [ ] Módulos ES6

## 🗒️ Notas pessoais


## 🧩 Conceitos relacionados

- [[Conceitos/HTTP|HTTP]]

## 🔗 Veja também

- [[Estudos/TypeScript|TypeScript]]
- [[Estudos/React|React]]
- [[Snippets/JavaScript|Snippets de JavaScript]]
