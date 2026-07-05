---
type: tech
area: Estudos
id: fw-nextjs-examples
category: Next.js
created: 2026-07-05
updated: 2026-07-05
---
# ⚡ Next.js - Code Examples

---

## 🛒 E-commerce Product Page

```javascript
// pages/product/[id].js
export async function getStaticProps({ params }) {
  const product = await fetch(`/api/product/${params.id}`).then(r => r.json());
  return { props: { product }, revalidate: 60 };
}

export async function getStaticPaths() {
  const products = await fetch('/api/products').then(r => r.json());
  return {
    paths: products.map(p => ({ params: { id: p.id } })),
    fallback: 'blocking'
  };
}

export default function Product({ product }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>${product.price}</p>
    </div>
  );
}
```

---

## 📊 Dashboard

Full-stack Next.js with API routes + DB.

---

## 🔐 Auth Example

NextAuth.js integration.

---

## 💬 Real-time Chat

WebSocket or Firebase integration.

---

**Status:** ✅ Completo
