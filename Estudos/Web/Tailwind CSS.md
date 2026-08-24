---
type: tech
area: Estudos
status: aprendendo
tecnologia: Tailwind CSS
tags:
  - tech
  - estudo
  - frontend
  - css
created: 2026-08-24
updated: 2026-08-24
---
# 🎨 Tailwind CSS

> [!tip] Status
> 🟢 Em uso

## 📝 Resumo

Framework CSS *utility-first*: em vez de escrever classes semânticas com regras próprias, você compõe a interface a partir de classes atômicas de uma só responsabilidade.

```html
<!-- CSS tradicional -->
<div class="card">...</div>
<style>.card { padding: 1rem; border-radius: .5rem; background: white; }</style>

<!-- Tailwind -->
<div class="p-4 rounded-lg bg-white shadow">...</div>
```

## 🧠 Conceitos principais

- **Utility-first** — cada classe faz uma coisa: `p-4` é padding, `flex` é display, `gap-2` é espaçamento
- **Escala de design embutida** — `p-1 p-2 p-4 p-8` seguem uma progressão; isso força consistência sem disciplina manual
- **Responsividade por prefixo** — `md:flex lg:grid`, aplicados a partir do breakpoint (mobile-first)
- **Estado por prefixo** — `hover:`, `focus:`, `disabled:`, `group-hover:`, `dark:`
- **Purge automático** — o build varre os arquivos e mantém só as classes usadas; o CSS final costuma ficar em poucos KB

## 💻 Instalação (v4, com Vite)

```bash
npm install tailwindcss @tailwindcss/vite
```

```ts
// vite.config.ts
import tailwindcss from "@tailwindcss/vite";
export default defineConfig({ plugins: [tailwindcss()] });
```

```css
/* index.css */
@import "tailwindcss";
```

> [!note] A v4 mudou a instalação
> Nas versões anteriores era preciso `tailwind.config.js`, PostCSS e as diretivas `@tailwind base/components/utilities`. Na v4, o plugin do Vite mais um `@import "tailwindcss"` bastam, e a configuração passou a ser feita em CSS com `@theme`.

## 🧩 Padrões úteis

```html
<!-- Layout responsivo -->
<div class="flex flex-col md:flex-row gap-4">

<!-- Centralização -->
<div class="flex items-center justify-center min-h-screen">

<!-- Card -->
<div class="rounded-2xl border border-gray-200 bg-white p-6 shadow-sm">

<!-- Balão de chat alinhado por condição -->
<div class="max-w-[75%] rounded-2xl px-4 py-2 bg-blue-500 text-white ml-auto">

<!-- Tema escuro -->
<div class="bg-white text-gray-900 dark:bg-gray-900 dark:text-gray-100">
```

## 🔀 Classes condicionais em React

```tsx
import clsx from "clsx";

<button className={clsx(
  "px-4 py-2 rounded-lg font-medium transition",
  ativo ? "bg-blue-600 text-white" : "bg-gray-100 text-gray-700",
  disabled && "opacity-50 cursor-not-allowed",
)}>
```

> [!warning] Nunca construa nome de classe por concatenação
> `` className={`text-${cor}-500`} `` **não funciona**: o purge é feito por varredura de texto, e a classe `text-red-500` nunca aparece literalmente no código. Use um mapa explícito:
> ```tsx
> const cores = { vermelho: "text-red-500", azul: "text-blue-500" };
> <span className={cores[cor]} />
> ```

## ⚖️ Prós e contras

| ✅ | ❌ |
|---|---|
| Sem inventar nome de classe | HTML fica visualmente carregado |
| Sem CSS morto acumulando | Curva inicial para decorar utilitários |
| Consistência pela escala | Repetição sem componentização |
| Estilo junto do markup | Difícil de ler em diff grande |
| Bundle final pequeno | Depende do build |

> [!tip] O contra-argumento do "HTML poluído" é componentização
> Repetir vinte classes em dez lugares é o problema. Extrair um componente `<Card>` que carrega essas classes uma vez resolve — e é o que você faria de qualquer forma em [[React|React]]. Em templates sem componentes, `@apply` cumpre o mesmo papel, com moderação.

## 🔗 Conceitos relacionados

- [[CSS|CSS]] · [[HTML|HTML]] · [[React|React]] · [[Vite|Vite]]

## Veja também

- [[Estudos|📚 Estudos]]
- [[Documentações|Documentações]]
