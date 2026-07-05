---
type: tech
area: Estudos
id: fw-svelte-animations
category: Svelte
created: 2026-07-05
updated: 2026-07-05
---
# 🔥 Svelte - Animations & Transitions

---

## 🎬 Transitions

```svelte
<script>
  import { fade } from 'svelte/transition';
  let visible = true;
</script>

{#if visible}
  <div transition:fade>Fades in and out</div>
{/if}

<button on:click={() => visible = !visible}>Toggle</button>
```

---

## 🎨 Animations

```svelte
<script>
  import { animate } from 'svelte/animate';
  let x = 0;
</script>

<div animate:animate={{ duration: 1000 }} style="transform: translateX({x}px)">
  Animate
</div>
```

---

## ⚡ Directives

Built-in: fade, slide, scale, draw, etc.

---

**Status:** ✅ Completo
