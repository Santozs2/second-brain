---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-divide-conquer
category: Algorithms
tags:
  - algorithms
created: 2026-07-05
updated: 2026-07-05
---
# ⚔️ Divide & Conquer

> Quebra problema em subproblemas independentes, resolve, e combina.

---

## 📖 Definição

**3 Etapas:**
1. **Divide:** Quebra em subproblemas menores
2. **Conquer:** Resolve subproblemas recursivamente
3. **Combine:** Combina soluções dos subproblemas

---

## 💻 Exemplos Clássicos

### 1. Merge Sort
```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    
    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    
    result.extend(left[i:])
    result.extend(right[j:])
    return result

# Complexidade: O(n log n)
# Espaço: O(n)
# Estável: Sim
```

### 2. Quick Sort
```python
def quick_sort(arr):
    if len(arr) <= 1:
        return arr
    
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    
    return quick_sort(left) + middle + quick_sort(right)

# Complexidade: O(n log n) médio, O(n²) pior
# Espaço: O(log n) recursão
# In-place: Sim (com partição)
# Estável: Não
```

### 3. Binary Search
```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    
    while left <= right:
        mid = (left + right) // 2
        
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return -1

# Complexidade: O(log n)
# Pré-requisito: Array DEVE estar sorted
```

### 4. Strassen's Matrix Multiplication
```python
# Multiplica matrizes 2x2 com 7 multiplicações
# Em vez de 8 (divide & conquer)
# Para matrizes grandes: O(n^2.807) vs O(n^3)
```

---

## 📊 Recorrência

```
T(n) = a*T(n/b) + f(n)

Master Theorem:
1. Se f(n) = O(n^(log_b(a)-ε))  → T(n) = O(n^(log_b(a)))
2. Se f(n) = Θ(n^(log_b(a)))    → T(n) = O(n^(log_b(a)) * log n)
3. Se f(n) = Ω(n^(log_b(a)+ε))  → T(n) = O(f(n))

Merge Sort: T(n) = 2T(n/2) + O(n)
           → Caso 2 → O(n log n)
```

---

## 🎯 Casos de Uso

✅ **Sorting:** Merge Sort, Quick Sort  
✅ **Busca:** Binary Search  
✅ **Problema de Inversão**  
✅ **Multiplicação de Matrizes**  
✅ **Casco Convexo**

---

## ❓ Entrevista

1. Implementar Merge Sort
2. Quick Sort (com partition)
3. Binary Search em rotated array
4. Encontrar elemento em rotated array
5. Contar inversões em array

---

## 🔗 Relacionado

- [[cs-sorting-algorithms|Sorting]]
- [[cs-search-algorithms|Searching]]
- [[cs-recurrence-relations|Recurrence Relations]]

---

**Status:** ✅ Completo
