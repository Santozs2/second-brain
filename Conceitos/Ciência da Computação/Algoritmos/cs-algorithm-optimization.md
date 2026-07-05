---
type: concept
area: Conceitos
difficulty: advanced
id: cs-algorithm-optimization
category: Algorithms
tags:
  - algorithms
created: 2026-07-05
updated: 2026-07-05
---
# ⚡ Otimização de Algoritmos

> Estratégias para reduzir tempo e/ou espaço de um algoritmo sem mudar o resultado.

---

## 1️⃣ Antes de otimizar

1. **Meça** — perfilamento (profiling) mostra o gargalo real
2. **Priorize** — otimize o trecho quente, não o irrelevante
3. **Cuide do Big O primeiro** — trocar O(n²) por O(n log n) vence qualquer micro-otimização

Ver [[cs-big-o|Big O]].

---

## 2️⃣ Técnicas por complexidade

| Técnica | Efeito típico |
|---------|---------------|
| Memoização / DP | Elimina recomputação (exponencial → polinomial) |
| Hashing | Busca O(n) → O(1) |
| Dois ponteiros / janela deslizante | O(n²) → O(n) |
| Busca binária | O(n) → O(log n) em dados ordenados |
| Pré-computação / prefix sums | Consultas O(n) → O(1) |
| Estruturas certas (heap, trie) | Melhora inserção/consulta |

---

## 3️⃣ Espaço vs Tempo

Trade-off central: gastar memória (caches, lookup tables) para ganhar velocidade — ou o contrário. Nem sempre dá para ter os dois.

---

## 4️⃣ Micro-otimizações

Depois do algoritmo correto: evitar alocações em loop, localidade de cache (ver [[cs-memory-hierarchy|Memory Hierarchy]]), reduzir chamadas de função. Ganhos menores, use só onde importa.

---

## ⚠️ Regra de ouro

> "Otimização prematura é a raiz de todo mal." — Knuth

Corretude e legibilidade primeiro; otimize com dados na mão.

---

## 🔗 Relacionado

- [[cs-algorithm-intro|Algorithm Intro]]
- [[cs-big-o|Big O]]
- [[cs-dynamic-programming|Dynamic Programming]]
- [[cs-amortized-analysis|Amortized Analysis]]

---

**Status:** ✅ Completo
