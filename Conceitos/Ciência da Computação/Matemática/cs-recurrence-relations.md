---
type: concept
id: cs-recurrence-relations
created: 2026-07-05
updated: 2026-07-05
category: Mathematics
tags:
  - type/concept
  - domain/mathematics
  - difficulty/advanced
---

# 🔄 Recurrence Relations

> Relação recursiva entre termos.

---

## 📝 Exemplos

```
Fibonacci:  T(n) = T(n-1) + T(n-2), T(1)=T(2)=1
Binary Search: T(n) = T(n/2) + O(1)
Merge Sort:    T(n) = 2T(n/2) + O(n)
```

---

## 🧮 Master Theorem

Para T(n) = aT(n/b) + f(n):

1. Se f(n) = O(n^(log_b(a)-ε)) → T(n) = O(n^(log_b(a)))
2. Se f(n) = Θ(n^(log_b(a))) → T(n) = O(n^(log_b(a)) log n)
3. Se f(n) = Ω(n^(log_b(a)+ε)) → T(n) = O(f(n))

---

**Status:** ✅ Completo
