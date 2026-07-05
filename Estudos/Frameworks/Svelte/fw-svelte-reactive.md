---
type: tech
area: Estudos
id: fw-svelte-reactive
category: Svelte
created: 2026-07-05
updated: 2026-07-05
---
# 🔥 Svelte - Reactivity

---

## 📝 Reactive Declarations

```svelte
<script>
  let count = 0;
  $: doubled = count * 2;
  $: console.log(count);
</script>

<p>{count} × 2 = {doubled}</p>
```

---

## 🔄 Reactive Statements

```svelte
$: if (count > 10) {
  alert('Big number!');
}
```

---

## 📊 Stores

```javascript
// store.js
import { writable } from 'svelte/store';
export const count = writable(0);
```

```svelte
<script>
  import { count } from './store.js';
</script>

<button on:click={() => $count++}>
  Count: {$count}
</button>
```

---

**Status:** ✅ Completo
