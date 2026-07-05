---
type: concept
id: cs-garbage-collection
created: 2026-07-05
updated: 2026-07-05
category: Operating Systems
tags:
  - type/concept
  - domain/operating-systems
  - difficulty/advanced
---

# 🗑️ Garbage Collection

> Recuperar memória de objetos não usados.

---

## 🔍 Algoritmos

### Mark & Sweep
```
1. MARK: Marca objetos acessíveis
2. SWEEP: Deleta não marcados
O(heap size), para completo
```

### Generational GC
```
Divide objetos por idade:
- Young generation: GC frequente (rápido)
- Old generation: GC raro (lento)
Maioria objetos morrem jovens
```

### Reference Counting
```
Cada objeto tem contador de referências
Quando zero, deleta
Problema: ciclos de referência
```

---

## ⚠️ Stop-the-World

Threads pausam durante GC - latência visível

---

## 🎯 Trade-offs

**Throughput:** Mais GC = menos throughput  
**Latency:** GC pause pode ser longo  
**Memory:** Overhead para tracking

---

**Status:** ✅ Completo
