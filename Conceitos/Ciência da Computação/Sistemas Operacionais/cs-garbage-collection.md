---
type: concept
area: Conceitos
difficulty: advanced
id: cs-garbage-collection
category: Operating Systems
tags:
  - operating-systems
created: 2026-07-05
updated: 2026-07-05
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

## 🔗 Ver também nesta área

- [[cs-disk-io]]
- [[cs-file-system]]
- [[cs-process]]
