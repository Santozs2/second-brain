---
type: tech
area: Estudos
id: fw-nextjs-overview
category: Next.js
created: 2026-07-05
updated: 2026-07-05
---
# ⚡ Next.js - Complete Framework Guide

---

## 📊 Overview

**Type:** React framework for production  
**Features:** SSR, SSG, API routes, optimized  
**Created:** 2016 (Vercel)  
**Status:** Production-ready  

---

## 🎯 Key Features

- **SSR (Server-Side Rendering)** - Render on server
- **SSG (Static Generation)** - Pre-render at build time
- **ISR (Incremental Static Regeneration)** - Update static pages
- **API Routes** - Backend in same project
- **Image Optimization** - Automatic image optimization
- **Code Splitting** - Automatic bundling
- **TypeScript** - Built-in support

---

## 🏗️ Project Structure

```
pages/
  index.js          → /
  about.js          → /about
  api/
    hello.js        → /api/hello
public/
lib/
components/
styles/
next.config.js
```

---

## 🚀 Rendering Modes

**SSR:** Render on each request
**SSG:** Render once at build time
**ISR:** Re-render periodically
**CSR:** Client-side rendering

---

## 📦 Ecosystem

- **Database:** Prisma, Mongoose
- **Auth:** NextAuth.js
- **Styling:** Tailwind, Emotion
- **Deployment:** Vercel, AWS

---

## 🔗 Relacionado

- [[Next.js|Nota principal — Next.js]]
- [[fw-react-overview|React (base do Next.js)]]

---

**Status:** ✅ Completo

## 🗂️ Neste guia

- [[fw-nextjs-api]]
- [[fw-nextjs-deploy]]
- [[fw-nextjs-examples]]
- [[fw-nextjs-performance]]
- [[fw-nextjs-rendering]]
- [[fw-nextjs-security]]
