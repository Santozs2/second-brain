---
type: tech
tags:
  - tech
  - estudo
  - frontend
  - backend
tecnologia: TypeScript
status: aprendendo
created: 2026-06-30
updated: 2026-06-30
---

# TypeScript

> [!tip] Status
> 🟡 Aprendendo

## 📝 Resumo

TypeScript é um superset do JavaScript que adiciona tipagem estática. Ajuda a evitar erros em tempo de desenvolvimento e melhora o autocomplete.

## 🧠 Conceitos principais

- **Tipos básicos**: `string`, `number`, `boolean`, `any`, `unknown`
- **Interfaces e types**: `interface`, `type`
- **Genéricos**: `function identidade<T>(valor: T): T`
- **Union e Intersection types**
- **Tipagem de funções e objetos**
- **`tsconfig.json`** e compilação

## 💻 Exemplos

```ts
interface Usuario {
  id: number;
  nome: string;
  ativo?: boolean;
}

function saudacao(usuario: Usuario): string {
  return `Olá, ${usuario.nome}!`;
}
```

## 🔗 Links úteis

- [Documentação oficial TypeScript](https://www.typescriptlang.org/docs/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)

## ✅ Checklist de aprendizado

- [x] Tipos básicos
- [ ] Interfaces vs Types
- [ ] Genéricos
- [ ] Tipagem em React (props, hooks)
- [ ] Configuração de `tsconfig.json`

## 🗒️ Notas pessoais


## 🔗 Veja também

- [[JavaScript|JavaScript]]
- [[React|React]]
