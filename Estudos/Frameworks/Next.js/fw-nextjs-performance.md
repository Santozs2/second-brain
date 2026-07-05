---
type: technology
id: fw-nextjs-performance
created: 2026-07-05
category: Next.js
---

# ⚡ Next.js - Performance

---

## 🖼️ Image Optimization

```javascript
import Image from 'next/image';

export default function Hero() {
  return (
    <Image
      src="/hero.jpg"
      alt="Hero"
      width={1200}
      height={600}
      priority
    />
  );
}
```

Automatic resizing, format conversion, lazy loading.

---

## 📦 Code Splitting

Automatic per-page splitting.

---

## 🔄 SWR Hook

```javascript
import useSWR from 'swr';

export function UserProfile({ userId }) {
  const { data, error } = useSWR(`/api/user/${userId}`, fetch);
  if (error) return <div>failed</div>;
  if (!data) return <div>loading</div>;
  return <div>{data.name}</div>;
}
```

---

## 📊 Metrics

- **FCP:** < 1.5s
- **LCP:** < 2.5s
- **CLS:** < 0.1

---

**Status:** ✅ Completo
