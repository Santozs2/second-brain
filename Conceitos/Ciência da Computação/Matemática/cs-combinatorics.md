---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-combinatorics
category: Mathematics
tags:
  - mathematics
created: 2026-07-05
updated: 2026-07-05
---
# 🔢 Combinatorics

> Contagem e arranjo de elementos.

---

## 📖 Definições

**Permutação:** Ordem importa  
P(n,r) = n! / (n-r)!

**Combinação:** Ordem NÃO importa  
C(n,r) = n! / (r! * (n-r)!)

---

## 💻 Exemplos

```python
from math import factorial

# Permutação: Arranjar 3 de 5
def permutation(n, r):
    return factorial(n) // factorial(n - r)

# Combinação: Escolher 3 de 5
def combination(n, r):
    return factorial(n) // (factorial(r) * factorial(n - r))

# Números: P(5,3) = 60, C(5,3) = 10

# Triângulo de Pascal (Binomial)
def pascal_triangle(n):
    for i in range(n):
        for j in range(i + 1):
            print(combination(i, j), end=' ')
        print()
```

---

## 📊 Aplicações

✅ **Probabilidade (maneiras de arranjar)**  
✅ **Análise combinatória**  
✅ **Criptografia (chaves)**  
✅ **Planejamento**

---

**Status:** ✅ Completo
