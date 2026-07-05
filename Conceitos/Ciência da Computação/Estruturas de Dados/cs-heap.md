---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-heap
category: Data Structures
tags:
  - data-structures
created: 2026-07-05
updated: 2026-07-05
---
# 📚 Heap (Amontoado)

> Árvore binária completa que satisfaz propriedade de heap.

---

## 📖 Definição

**Heap**:
- **Árvore binária completa:** Todos níveis preenchidos, último parcial esquerda
- **Min Heap:** Pai < Filhos
- **Max Heap:** Pai > Filhos
- **Operações:** Insert/Delete O(log n), Get Min O(1)

---

## 🧠 Estrutura

```
Max Heap:
       50
      /  \
    30    20
   / \   /
  10  5 15

Array: [50, 30, 20, 10, 5, 15]

Parent(i) = (i-1) // 2
Left(i) = 2*i + 1
Right(i) = 2*i + 2
```

---

## 💻 Implementação

```python
class MinHeap:
    def __init__(self):
        self.heap = []
    
    def parent(self, i):
        return (i - 1) // 2
    
    def left_child(self, i):
        return 2 * i + 1
    
    def right_child(self, i):
        return 2 * i + 2
    
    def insert(self, key):
        self.heap.append(key)
        self._bubble_up(len(self.heap) - 1)
    
    def _bubble_up(self, i):
        while i > 0 and self.heap[self.parent(i)] > self.heap[i]:
            self.heap[i], self.heap[self.parent(i)] = \
                self.heap[self.parent(i)], self.heap[i]
            i = self.parent(i)
    
    def extract_min(self):
        if not self.heap:
            return None
        min_val = self.heap[0]
        self.heap[0] = self.heap[-1]
        self.heap.pop()
        self._bubble_down(0)
        return min_val
    
    def _bubble_down(self, i):
        smallest = i
        left = self.left_child(i)
        right = self.right_child(i)
        
        if left < len(self.heap) and self.heap[left] < self.heap[smallest]:
            smallest = left
        if right < len(self.heap) and self.heap[right] < self.heap[smallest]:
            smallest = right
        
        if smallest != i:
            self.heap[i], self.heap[smallest] = \
                self.heap[smallest], self.heap[i]
            self._bubble_down(smallest)
```

---

## 📊 Operações

| Operação | Tempo |
|----------|-------|
| Insert | O(log n) |
| Extract Min | O(log n) |
| Get Min | O(1) |
| Heapify (construir) | O(n) |

---

## 🎯 Casos de Uso

✅ **Priority Queue**  
✅ **Dijkstra's Algorithm**  
✅ **Heap Sort**  
✅ **Median Finding**  
✅ **Top K Elements**

---

## 💡 Heapify

```python
# Build heap from array em O(n)
def heapify(arr):
    for i in range(len(arr) // 2 - 1, -1, -1):
        bubble_down(arr, i, len(arr))
```

---

## ❓ Entrevista

1. Implementar Min Heap
2. Kth Largest Element
3. Merge K Sorted Lists
4. Top K Frequent Elements
5. Sliding Window Maximum

---

## 🔗 Relacionado

- [[cs-tree|Tree]]
- [[cs-sorting-algorithms|Heap Sort]]
- [[cs-dijkstra|Dijkstra]]

---

**Status:** ✅ Completo
