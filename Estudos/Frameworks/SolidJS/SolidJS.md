---
type: tech
area: Estudos
status: explorar
tecnologia: SolidJS
tags:
  - tech
  - estudo
  - frontend
created: 2026-07-05
updated: 2026-07-05
---
# SolidJS

> [!tip] Status
> 🔵 Explorar

## 📝 Resumo

SolidJS é uma biblioteca reativa com sintaxe parecida com React (JSX), mas sem Virtual DOM. Usa **signals** de fina granularidade: só o que depende de um valor é reexecutado quando ele muda.

## 🧠 Conceitos principais

- **Signals**: `createSignal` (getter/setter reativos)
- **Efeitos**: `createEffect`, `createMemo`
- **Componentes** executam uma única vez (sem re-render)
- **Stores** para estado aninhado
- **Roteamento** com `@solidjs/router`

## 💻 Exemplo

```jsx
import { createSignal } from "solid-js";

function Contador() {
  const [contagem, setContagem] = createSignal(0);
  return (
    <button onClick={() => setContagem(contagem() + 1)}>
      Cliques: {contagem()}
    </button>
  );
}
```

## 🔗 Links úteis

- [Documentação oficial SolidJS](https://www.solidjs.com/)

## 📖 Aprofundar

- [[fw-solidjs-overview|Guia detalhado de SolidJS]] — signals, componentes, forms, routing e deploy

## 🔗 Veja também

- [[React|React]]
- [[JavaScript|JavaScript]]
