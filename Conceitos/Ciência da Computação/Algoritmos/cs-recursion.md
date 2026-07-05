---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-recursion
category: Algorithms
tags:
  - algorithms
created: 2026-07-05
updated: 2026-07-05
---
# 🔁 Recursão

> Uma função que resolve um problema chamando a si mesma em subproblemas menores.

---

## 1️⃣ Anatomia

Toda recursão precisa de:

- **Caso base** — condição que encerra as chamadas
- **Caso recursivo** — chamada a si mesma aproximando-se do caso base

```python
def fatorial(n):
    if n <= 1:          # caso base
        return 1
    return n * fatorial(n - 1)  # caso recursivo
```

---

## 2️⃣ Pilha de Chamadas

Cada chamada empilha um frame na call stack. Recursão muito profunda causa **stack overflow**.

```python
# Python limita a ~1000 níveis por padrão
import sys
sys.setrecursionlimit(10000)
```

---

## 3️⃣ Tipos

| Tipo | Descrição |
|------|-----------|
| Linear | Uma chamada por passo (fatorial) |
| Binária | Duas chamadas (Fibonacci ingênuo) |
| De cauda | Chamada recursiva é a última operação |
| Mútua | `a()` chama `b()` que chama `a()` |

---

## 4️⃣ Recursão vs Iteração

- **Recursão**: código mais expressivo para estruturas recursivas (árvores, grafos, divide & conquer)
- **Iteração**: menor overhead de memória, sem risco de stack overflow

Muitas recursões podem virar loops; recursão de cauda pode ser otimizada (nem toda linguagem faz isso).

---

## 5️⃣ Armadilha: Recomputação

Fibonacci ingênuo recalcula os mesmos valores — O(2ⁿ). A solução é **memoização** (ver Programação Dinâmica).

```python
from functools import cache

@cache
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)
```

---

## ❓ Entrevista

1. Inverter uma string / lista recursivamente
2. Percursos de árvore (pré/in/pós-ordem)
3. Torres de Hanói
4. Backtracking (permutações, N-Rainhas)

---

## 🔗 Relacionado

- [[cs-dynamic-programming|Dynamic Programming]]
- [[cs-divide-conquer|Divide & Conquer]]
- [[cs-tree|Tree]]
- [[cs-stack|Stack]]

---

**Status:** ✅ Completo
