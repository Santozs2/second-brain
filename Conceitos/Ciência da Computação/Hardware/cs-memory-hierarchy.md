---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-memory-hierarchy
category: Hardware
tags:
  - hardware
created: 2026-07-05
updated: 2026-07-05
---
# 🏛️ Memory Hierarchy

> Piramide de velocidade vs tamanho.

---

## 📊 Hierarchy

```
┌─────────────────┬───────────┬──────────┐
│ Level           │ Tamanho   │ Latência │
├─────────────────┼───────────┼──────────┤
│ Registradores   │ 1 KB      │ 1 ns     │
│ L1 Cache        │ 64 KB     │ 4 ns     │
│ L2 Cache        │ 256 KB    │ 10 ns    │
│ L3 Cache        │ 8 MB      │ 50 ns    │
│ RAM             │ 16 GB     │ 100 ns   │
│ SSD             │ 500 GB    │ 0.1 ms   │
│ HDD             │ 2 TB      │ 10 ms    │
└─────────────────┴───────────┴──────────┘
```

---

## 💡 Localidade

**Temporal:** Dados recentemente usados serão usados novamente  
**Spatial:** Dados perto são usados juntos

Cache explora ambas!

---

## 🎯 Implicações

1. **Array Layout:** Row vs Column major importa
2. **Algoritmos:** Minimizar memory misses
3. **CPU Bound:** Latência cache crítica

---

**Status:** ✅ Completo
