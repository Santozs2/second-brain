---
type: concept
area: Conceitos
difficulty: beginner
id: cs-asymptotic-analysis
category: Complexity
tags:
  - complexity
created: 2026-07-05
updated: 2026-07-05
---
# 📐 Asymptotic Analysis

> Notação matemática para descrever comportamento de funções.

---

## 📖 Definições

### Big O (Upper Bound)
```
f(n) = O(g(n)) se ∃ c, n₀ : f(n) ≤ c*g(n) ∀n ≥ n₀

Significado: f(n) não cresce mais rápido que g(n)
Pior caso
```

### Big Omega (Lower Bound)
```
f(n) = Ω(g(n)) se ∃ c, n₀ : f(n) ≥ c*g(n) ∀n ≥ n₀

Significado: f(n) cresce pelo menos tão rápido quanto g(n)
Melhor caso
```

### Big Theta (Tight Bound)
```
f(n) = Θ(g(n)) se f(n) = O(g(n)) E f(n) = Ω(g(n))

Significado: f(n) e g(n) crescem no mesmo ritmo
Caso médio
```

### Little o (Estritamente menor)
```
f(n) = o(g(n)) se lim(n→∞) f(n)/g(n) = 0

f(n) cresce estritamente mais lentamente
```

---

## 🔄 Propriedades

```
Transitívidade: f = O(g) e g = O(h) → f = O(h)
Soma: O(f) + O(g) = O(max(f, g))
Produto: O(f) * O(g) = O(f * g)
Constante: O(c*f) = O(f)
```

---

## 📊 Comparação

```
n!  >  2^n  >  n³  >  n²  >  n log n  >  n  >  log n  >  1

n=10:       3.6M > 1024 > 1000 > 100 > 33 > 10 > 3 > 1
n=100:      Huge > Huge > 1M   > 10K > 664 > 100 > 6 > 1
n=1000:     ...  > ...  > 1B   > 1M  > 9965 > 1000 > 9 > 1
```

---

## 💡 Análise Prática

### Análise de Tempo

```python
# O(1)
def first_element(arr):
    return arr[0]

# O(n)
def linear_search(arr, target):
    for x in arr:
        if x == target:
            return True
    return False

# O(n²)
def bubble_sort(arr):
    for i in range(len(arr)):
        for j in range(len(arr) - 1):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]

# O(log n)
def binary_search(arr, target):
    # Ver [[cs-search-algorithms]]

# O(n log n)
def merge_sort(arr):
    # Ver [[cs-sorting-algorithms]]
```

### Análise de Espaço

```python
# O(1) - espaço constante
def sum_numbers(arr):
    total = 0
    for x in arr:
        total += x
    return total

# O(n) - espaço linear
def copy_array(arr):
    return arr[:]  # Cópia

# O(n) - stack profundidade
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n-1)  # Recursão profunda
```

---

## 🎯 Regras de Ouro

```
1. Ignore constantes: O(2n) = O(n)
2. Termos dominantes: O(n² + n) = O(n²)
3. Múltiplos loops: O(n) * O(m) = O(n*m)
4. Aninhados: O(n²) para loops aninhados
5. Recursão: Use relação de recorrência ou Master Theorem
```

---

## 📊 Exemplos

```
for i in range(n):          → O(n)
    for j in range(n):      → O(n²)
        
for i in range(n):          → O(n)
    for j in range(10):     → O(10n) = O(n)

for i in range(n):          → O(n + m)
for j in range(m):

i = 1
while i < n:                → O(log n)
    i *= 2
```

---

## 🔗 Relacionado

- [[cs-big-o|Big O Notation (detalhado)]]
- [[cs-amortized-analysis|Amortized Analysis]]
- [[cs-algorithm-intro|Algoritmos]]

---

**Status:** ✅ Completo
