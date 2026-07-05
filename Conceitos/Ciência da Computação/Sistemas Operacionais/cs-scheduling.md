---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-scheduling
category: Operating Systems
tags:
  - operating-systems
created: 2026-07-05
updated: 2026-07-05
---
# ⏰ CPU Scheduling

> Decidir qual processo executa quando.

---

## 🎯 Algoritmos

### FCFS (First Come First Served)
```
Simples mas Convoy effect (longo espera curtos)
```

### SJF (Shortest Job First)
```
Melhor tempo médio
Problema: Precisa saber duração
```

### Round Robin
```
Cada processo: time slice (quantum)
Justo, bom para interativo
```

### Priority Queue
```
Alta prioridade → executa primeiro
Problema: Starvation (baixa prioridade nunca executa)
Solução: Aging (aumenta prioridade com tempo)
```

### Multilevel Feedback Queue
```
Múltiplas filas com diferentes prioridades
Processos interativos: fila alta
Batch: fila baixa
```

---

## 📊 Métricas

- **Turnaround time:** Entrada até saída
- **Wait time:** Tempo esperando na fila
- **Response time:** Até primeira execução
- **Throughput:** Processos por unidade tempo

---

**Status:** ✅ Completo
