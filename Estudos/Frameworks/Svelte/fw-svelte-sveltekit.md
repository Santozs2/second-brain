---
type: technology
id: fw-svelte-sveltekit
created: 2026-07-05
category: Svelte
---

# 🔥 Svelte - SvelteKit

---

## 🚀 Full-Stack Framework

SvelteKit = Svelte + SSR + Routing

---

## 📂 Routes

```
src/routes/
  +page.svelte       → /
  +page.server.js    → Load data
  [id]/+page.svelte  → /[id]
  api/hello/+server.js → API endpoint
```

---

## 📊 Data Loading

```javascript
// +page.server.js
export async function load({ params }) {
  return { posts: [] };
}
```

---

## 🔧 API Routes

```javascript
// api/users/+server.js
export async function GET() {
  return new Response(JSON.stringify([]));
}
```

---

**Status:** ✅ Completo
