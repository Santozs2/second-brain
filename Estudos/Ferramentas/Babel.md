---
type: tech
area: Estudos
status: explorar
tecnologia: Babel
tags:
  - tech
  - estudo
  - ferramenta
created: 2026-07-05
updated: 2026-07-05
---
# Babel

> [!tip] Status
> 🔵 Explorar

## 📝 Resumo

Babel é um **transpilador** de JavaScript: converte código moderno (ES2015+) e sintaxes como JSX/TypeScript em JavaScript compatível com navegadores e ambientes mais antigos.

## 🧠 Conceitos principais

- **Presets**: conjuntos de plugins (`@babel/preset-env`, `preset-react`)
- **Plugins**: transformações pontuais de sintaxe
- **Polyfills**: adicionam APIs ausentes (via `core-js`)
- **`.babelrc` / `babel.config.js`**: configuração
- **Targets / browserslist**: define quais ambientes suportar

## 💻 Exemplo

```js
// babel.config.js
module.exports = {
  presets: [
    ["@babel/preset-env", { targets: "> 0.25%, not dead" }],
    "@babel/preset-react",
  ],
};
```

```js
// entrada (moderno)      → saída (compatível)
const f = () => 1;        // var f = function () { return 1; };
```

## 🔗 Links úteis

- [Documentação oficial Babel](https://babeljs.io/)

## 🔗 Veja também

- [[Webpack|Webpack]]
- [[JavaScript|JavaScript]]
- [[TypeScript|TypeScript]]
