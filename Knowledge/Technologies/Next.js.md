---
type: tech
tags:
  - tech
  - estudo
  - frontend
tecnologia: Next.js
status: aprendendo
created: 2026-06-30
updated: 2026-06-30
---

# Next.js

> [!tip] Status
> 🟡 Aprendendo

## 📝 Resumo

Next.js é um framework React para produção, com roteamento baseado em arquivos, renderização no servidor (SSR), geração estática (SSG) e API routes.

## 🧠 Conceitos principais

- **App Router**: estrutura de pastas `app/`, `page.tsx`, `layout.tsx`
- **Renderização**: Server Components vs Client Components
- **Data fetching**: `fetch` com cache, `generateStaticParams`
- **API Routes / Route Handlers**
- **Otimizações**: `next/image`, `next/font`
- **Deploy** (geralmente na Vercel)

## 💻 Exemplos

```tsx
// app/page.tsx
export default async function Home() {
  const res = await fetch("https://api.exemplo.com/posts");
  const posts = await res.json();

  return (
    <main>
      {posts.map((post: any) => (
        <h2 key={post.id}>{post.titulo}</h2>
      ))}
    </main>
  );
}
```

## 🔗 Links úteis

- [Documentação oficial Next.js](https://nextjs.org/docs)
- [Learn Next.js](https://nextjs.org/learn)

## ✅ Checklist de aprendizado

- [ ] App Router e estrutura de pastas
- [ ] Server vs Client Components
- [ ] Data fetching e cache
- [ ] Route Handlers (API)
- [ ] Deploy na Vercel

## 🗒️ Notas pessoais


## 🔗 Veja também

- [[Estudos/React|React]]
- [[Estudos/TypeScript|TypeScript]]
