---
type: technology
id: fw-nextjs-rendering
created: 2026-07-05
category: Next.js
---

# ⚡ Next.js - Rendering Strategies

---

## 📊 SSR (Server-Side Rendering)

```javascript
// pages/posts/[id].js
export async function getServerSideProps(context) {
  const { id } = context.params;
  const post = await fetch(`/api/posts/${id}`).then(r => r.json());
  return { props: { post } };
}

export default function Post({ post }) {
  return <div>{post.title}</div>;
}
```

Rendered on each request.

---

## 📦 SSG (Static Generation)

```javascript
export async function getStaticProps(context) {
  const posts = await fetch('/api/posts').then(r => r.json());
  return { props: { posts }, revalidate: 3600 };
}

export async function getStaticPaths() {
  return { paths: [], fallback: 'blocking' };
}
```

Generated at build time.

---

## 🔄 ISR (Incremental Static Regeneration)

Revalidate: 3600 → Regenerate every hour.

---

## ⚡ CSR (Client-Side Rendering)

Regular React component, no special props needed.

---

**Status:** ✅ Completo
