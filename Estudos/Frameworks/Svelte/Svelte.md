---
type: tech
area: Estudos
status: explorar
tecnologia: Svelte
tags:
  - tech
  - estudo
  - frontend
created: 2026-07-05
updated: 2026-07-05
---
# Svelte

> [!tip] Status
> 🔵 Explorar

## 📝 Resumo

Svelte é um compilador de UI: em vez de rodar um Virtual DOM em runtime, ele compila os componentes em JavaScript otimizado no build. Resultado: bundles menores e reatividade sem boilerplate.

## 🧠 Conceitos principais

- **Reatividade por atribuição**: `let count = 0` já é reativo
- **Declarações reativas**: `$: dobro = count * 2`
- **Props**: `export let nome`
- **Stores**: estado compartilhado (`writable`, `readable`)
- **Eventos** e **bindings** (`bind:value`)
- **SvelteKit**: framework full-stack

## 💻 Exemplo

```svelte
<script>
  let contagem = 0;
</script>

<button on:click={() => contagem++}>
  Cliques: {contagem}
</button>
```

## 🔗 Links úteis

- [Documentação oficial Svelte](https://svelte.dev/)

## 📖 Aprofundar

- [[fw-svelte-overview|Guia detalhado de Svelte]] — reatividade, eventos, SvelteKit e testes

## 🔗 Veja também

- [[JavaScript|JavaScript]]
- [[Vue|Vue]]
