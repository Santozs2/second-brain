---
type: tech
area: Estudos
status: explorar
tecnologia: Nuxt
tags:
  - tech
  - estudo
  - frontend
created: 2026-07-05
updated: 2026-07-05
---
# Nuxt

> [!tip] Status
> 🔵 Explorar

## 📝 Resumo

Nuxt é o meta-framework para Vue, análogo ao Next.js para React. Adiciona roteamento por arquivos, renderização no servidor (SSR/SSG), auto-imports e uma camada de servidor (Nitro).

## 🧠 Conceitos principais

- **Roteamento por arquivos**: pasta `pages/`
- **Renderização**: SSR, SSG e híbrido
- **Auto-imports**: componentes e composables
- **Composables**: `useFetch`, `useState`, `useAsyncData`
- **Server routes** com Nitro (`server/api/`)
- **Middleware** de rota

## 💻 Exemplo

```vue
<script setup>
const { data: posts } = await useFetch("/api/posts");
</script>

<template>
  <h2 v-for="post in posts" :key="post.id">{{ post.titulo }}</h2>
</template>
```

## 🔗 Links úteis

- [Documentação oficial Nuxt](https://nuxt.com/)

## 📖 Aprofundar

- [[fw-nuxt-overview|Guia detalhado de Nuxt]] — SSR, composables, auto-routing, middleware e deploy

## 🔗 Veja também

- [[Vue|Vue]]
- [[Next.js|Next.js]]
