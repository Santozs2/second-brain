---
type: concept
id: cs-greedy-algorithms
created: 2026-07-05
updated: 2026-07-05
category: Algorithms
tags:
  - type/concept
  - domain/algorithms
  - difficulty/intermediate
---

# 🤑 Greedy Algorithms

> Escolhe melhor opção LOCAL esperando obter ótimo GLOBAL.

---

## 📖 Definição

**Greedy** não funciona sempre, mas quando funciona é rápido:
- **Choice Property:** Escolha local ótima leva ao ótimo global
- **Optimal Substructure:** Solução ótima contém soluções ótimas de subproblemas
- **Nunca reconsideração:** Após escolher, não volta atrás

---

## 💻 Exemplos Clássicos

### 1. Activity Selection
```python
def activity_selection(activities):
    # Atividades = (start, end)
    # Maximizar número de atividades sem sobreposição
    
    # Ordena por tempo fim
    activities.sort(key=lambda x: x[1])
    
    selected = [activities[0]]
    last_end = activities[0][1]
    
    for start, end in activities[1:]:
        if start >= last_end:  # Não sobrepõe
            selected.append((start, end))
            last_end = end
    
    return selected

# Complexidade: O(n log n) sorting
# Prova: Greedy choice é ótimo aqui
```

### 2. Huffman Coding
```python
import heapq

class Node:
    def __init__(self, char, freq):
        self.char = char
        self.freq = freq
        self.left = None
        self.right = None
    
    def __lt__(self, other):
        return self.freq < other.freq

def huffman_coding(chars, freqs):
    heap = [Node(c, f) for c, f in zip(chars, freqs)]
    heapq.heapify(heap)
    
    while len(heap) > 1:
        left = heapq.heappop(heap)
        right = heapq.heappop(heap)
        
        merged = Node(None, left.freq + right.freq)
        merged.left = left
        merged.right = right
        
        heapq.heappush(heap, merged)
    
    return heap[0]

# Uso: Compressão de dados (ZIP, JPEG)
# Complexidade: O(n log n)
```

### 3. Fractional Knapsack
```python
def fractional_knapsack(weights, values, capacity):
    # Items podem ser quebrados (diferente de 0/1)
    
    # Calcular value/weight ratio
    items = [(v/w, v, w) for v, w in zip(values, weights)]
    items.sort(reverse=True)  # Melhor ratio primeiro
    
    total_value = 0
    
    for ratio, value, weight in items:
        if capacity >= weight:
            total_value += value
            capacity -= weight
        else:
            # Pega fração do último item
            total_value += (capacity / weight) * value
            break
    
    return total_value

# Complexidade: O(n log n)
# Nota: 0/1 Knapsack NÃO é greedy
```

### 4. Dijkstra (Greedy Graph Algorithm)
```python
# Ver [[cs-graph-algorithms|Graph Algorithms]]
# Sempre escolhe distância mínima conhecida
```

---

## 🎯 Quando Greedy Funciona

| Problema | Greedy? |
|----------|---------|
| Activity Selection | ✅ SIM |
| Huffman Coding | ✅ SIM |
| Fractional Knapsack | ✅ SIM |
| 0/1 Knapsack | ❌ NÃO |
| Coin Change (certos) | ✅ SIM |
| Coin Change (geral) | ❌ NÃO |
| MST (Prim/Kruskal) | ✅ SIM |

---

## 📊 Características

```
Vantagem: Geralmente O(n log n) ou O(n)
Desvantagem: Nem sempre ótimo

Se Greedy falha:
- Tentar Dynamic Programming
- Ou backtracking
```

---

## 🎯 Problemas

✅ **Activity Selection**  
✅ **Huffman Coding**  
✅ **Dijkstra's Algorithm**  
✅ **MST (Minimum Spanning Tree)**  
✅ **Job Scheduling**

---

## ❓ Entrevista

1. Activity Selection problem
2. Quando Greedy funciona vs não funciona
3. Difference Knapsack vs Activity
4. Job scheduling with deadlines
5. Huffman coding

---

## 🔗 Relacionado

- [[cs-dynamic-programming|DP]]
- [[cs-graph-algorithms|Graph Algorithms]]
- [[cs-divide-conquer|Divide & Conquer]]

---

**Status:** ✅ Completo
