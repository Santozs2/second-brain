---
type: concept
area: Conceitos
difficulty: advanced
id: cs-gpu
category: Hardware
tags:
  - hardware
created: 2026-07-05
updated: 2026-07-05
---
# 🎮 GPU (Graphics Processing Unit)

> Paralelo massivo - 1000+ cores vs 8-16 cores CPU.

---

## 📊 Arquitetura

```
CPU: 8-16 cores rápidos, uso geral
GPU: 1000+ cores lentos, especializado

Cada core tem: próprio ALU, cache L1
Warp/Wavefront: 32 threads executam juntas
```

---

## 🔄 Execução

```
CPU: Minimizar latência (1-2 instruções)
GPU: Maximizar throughput (1000 instruções)
     Latência é escondida por 1000 threads
```

---

## 🎯 Aplicações

- **Gráficos:** Rendering (processamento paralelo)
- **ML/AI:** Tensor operations
- **Simulação:** Physics
- **Cripto:** Mining

---

## 🔗 CUDA/OpenCL

Programação paralela em GPU

---

**Status:** ✅ Completo
