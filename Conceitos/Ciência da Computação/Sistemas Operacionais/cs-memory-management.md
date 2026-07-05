---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-memory-management
category: Operating Systems
tags:
  - operating-systems
created: 2026-07-05
updated: 2026-07-05
---
# 🧠 Memory Management

> Como SO gerencia memória dos processos.

---

## 🏗️ Estratégias de Alocação

### First Fit
```
Primeiro espaço grande o bastante
Rápido, mas fragmentação
```

### Best Fit
```
Espaço que sobra menos
Mais lento, menos fragmentação
```

### Worst Fit
```
Espaço maior que necessário
Raro, ruim
```

---

## 📊 Fragmentação

**Interna:** Espaço dentro alocação não usado  
**Externa:** Espaço entre alocações fragmentado

---

## 🔧 Paginação vs Segmentação

### Paginação
```
Memória dividida em pages (4KB cada)
Endereço = Page Number + Offset
Sem fragmentação externa
```

### Segmentação
```
Divisão lógica (código, dados, stack)
Fragmentação externa possível
```

---

## 🎯 Substituição (Paging)

- **FIFO:** Primeiro que entrou, sai primeiro
- **LRU:** Menos recentemente usado
- **LFU:** Menos frequentemente usado
- **Ótimo:** Remove que será usado por último

---

**Status:** ✅ Completo

## 🔗 Ver também nesta área

- [[cs-disk-io]]
- [[cs-file-system]]
- [[cs-garbage-collection]]
