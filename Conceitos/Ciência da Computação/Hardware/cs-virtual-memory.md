---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-virtual-memory
category: Hardware
tags:
  - hardware
created: 2026-07-05
updated: 2026-07-05
---
# 💾 Virtual Memory

> Ilusão de memória maior usando disco.

---

## 🎯 Conceito

CPU vê endereço lógico → MMU (Memory Management Unit) converte → endereço físico

Se página não está em RAM → Page Fault → SO carrega do disco

---

## 🔄 Paginação

```
Memória lógica: 64 KB
Memória física: 32 KB
Disco: 1 GB

Página = 4 KB
Páginas lógicas: 16
Frames físicos: 8
```

---

## 📊 Page Table

```
Lógico → Físico (mapeamento)
Entrada: Valid bit, Frame number, Protection bits
```

---

## ⚠️ Thrashing

Muitos page faults → Mais tempo no disco que CPU  
Solução: aumentar RAM ou reduzir working set

---

**Status:** ✅ Completo
