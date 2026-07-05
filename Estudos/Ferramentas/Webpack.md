---
type: tech
area: Estudos
status: explorar
tecnologia: Webpack
tags:
  - tech
  - estudo
  - ferramenta
created: 2026-07-05
updated: 2026-07-05
---
# Webpack

> [!tip] Status
> 🔵 Explorar

## 📝 Resumo

Webpack é um **module bundler**: analisa o grafo de dependências do projeto (JS, CSS, imagens) e gera bundles otimizados para o navegador.

## 🧠 Conceitos principais

- **Entry / Output**: ponto de partida e arquivos gerados
- **Loaders**: transformam arquivos (ex.: `babel-loader`, `css-loader`)
- **Plugins**: estendem o build (`HtmlWebpackPlugin`, `DefinePlugin`)
- **Code splitting**: divide bundles para carregar sob demanda
- **Tree shaking**: remove código morto
- **Dev server** com hot reload

## 💻 Exemplo

```js
// webpack.config.js
module.exports = {
  entry: "./src/index.js",
  output: { filename: "bundle.js" },
  module: {
    rules: [{ test: /\.js$/, use: "babel-loader" }],
  },
};
```

## 🔗 Links úteis

- [Documentação oficial Webpack](https://webpack.js.org/)

## 🔗 Veja também

- [[Babel|Babel]]
- [[NPM|NPM]]
- [[JavaScript|JavaScript]]
