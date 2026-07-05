---
type: concept
id: cs-memory-management
created: 2026-07-05
updated: 2026-07-05
category: Operating Systems
tags:
  - type/concept
  - domain/operating-systems
  - difficulty/intermediate
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
