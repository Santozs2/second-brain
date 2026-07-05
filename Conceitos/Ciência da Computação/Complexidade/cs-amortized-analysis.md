---
type: concept
area: Conceitos
difficulty: advanced
id: cs-amortized-analysis
category: Complexity
tags:
  - complexity
created: 2026-07-05
updated: 2026-07-05
---
# ⚖️ Análise Amortizada

> Custo médio por operação ao longo de uma sequência, mesmo que operações isoladas sejam caras.

---

## 💡 Ideia

Algumas operações são raramente caras e frequentemente baratas. A análise amortizada distribui o custo alto entre as muitas operações baratas, dando um custo médio realista — diferente do pior caso pontual.

**Exemplo clássico:** adicionar em um array dinâmico (`list.append`). Quase sempre O(1); ocasionalmente O(n) quando precisa redimensionar. O custo **amortizado** é O(1).

---

## 🧮 Três Métodos

### 1. Agregado
Soma o custo total de *n* operações e divide por *n*.

### 2. Contábil (banker's)
Cobra um "crédito" extra nas operações baratas para pagar as caras depois.

### 3. Potencial
Define uma função de potencial Φ que mede "energia armazenada" na estrutura; o custo amortizado é o custo real ± variação de Φ.

---

## 📊 Exemplo: Array Dinâmico

Ao dobrar a capacidade, *n* inserções custam no total ~2n cópias:

```
custo total ≈ n (inserções) + n (cópias no total) = O(n)
amortizado por operação = O(n)/n = O(1)
```

---

## ⚠️ Não confunda

- **Amortizado** ≠ **caso médio**: amortizado é garantido sobre a sequência (sem probabilidade); caso médio assume distribuição de entradas.

---

## 🔗 Relacionado

- [[cs-asymptotic-analysis|Asymptotic Analysis]]
- [[cs-big-o|Big O]]
- [[cs-array|Array]]
- [[cs-hash-table|Hash Table]]

---

**Status:** ✅ Completo
