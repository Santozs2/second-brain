---
type: tech
area: Estudos
status: explorar
tecnologia: Vue
tags:
  - tech
  - estudo
  - frontend
created: 2026-07-05
updated: 2026-07-05
---
# Vue

> [!tip] Status
> 🔵 Explorar

## 📝 Resumo

Vue é um framework progressivo para construir interfaces. Combina template declarativo, reatividade e componentes de arquivo único (`.vue`), sendo fácil de adotar de forma incremental.

## 🧠 Conceitos principais

- **Reatividade**: `ref`, `reactive`, `computed`, `watch`
- **Componentes SFC**: `<template>`, `<script setup>`, `<style scoped>`
- **Diretivas**: `v-if`, `v-for`, `v-model`, `v-bind`, `v-on`
- **Props e emits**: comunicação pai ↔ filho
- **Ciclo de vida**: `onMounted`, `onUnmounted`
- **Vue Router** e **Pinia** (estado)

## 💻 Exemplo

```vue
<script setup>
import { ref } from "vue";
const contagem = ref(0);
</script>

<template>
  <button @click="contagem++">Cliques: {{ contagem }}</button>
</template>
```

## 🔗 Links úteis

- [Documentação oficial Vue](https://vuejs.org/)

## 📖 Aprofundar

- [[fw-vue-overview|Guia detalhado de Vue]] — componentes, reatividade, lifecycle e router

## 🔗 Veja também

- [[JavaScript|JavaScript]]
- [[TypeScript|TypeScript]]
- [[Nuxt|Nuxt]]
