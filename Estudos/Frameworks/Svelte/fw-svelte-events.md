---
type: tech
area: Estudos
id: fw-svelte-events
category: Svelte
created: 2026-07-05
updated: 2026-07-05
---
# 🔥 Svelte - Events & Binding

---

## 📌 Event Handling

```svelte
<button on:click={handleClick}>Click</button>
<input on:change={handleChange}>
<div on:mouseenter={handleHover}>Hover</div>
```

---

## 🔗 Two-Way Binding

```svelte
<script>
  let name = '';
</script>

<input bind:value={name}>
<p>{name}</p>
```

---

## 🎯 Event Modifiers

```svelte
<button on:click|once={handler}>Once</button>
<button on:click|preventDefault={handler}>Prevent</button>
```

---

**Status:** ✅ Completo
