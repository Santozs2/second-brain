---
type: concept
id: cs-queue
created: 2026-07-04
updated: 2026-07-04
category: Data Structures
tags:
  - type/concept
  - domain/data-structures
  - difficulty/beginner
---

# 🚗 Queue (Fila)

> FIFO - First In First Out. Como fila de banco.

---

## 📖 Definição

**Queue** é FIFO:
- **Enqueue:** Adiciona no final
- **Dequeue:** Remove do início
- **Tempo:** O(1) para todas ops

---

## 💻 Implementação

```python
from collections import deque

class Queue:
    def __init__(self):
        self.items = deque()
    
    def enqueue(self, item):
        self.items.append(item)
    
    def dequeue(self):
        return self.items.popleft() if self.items else None
    
    def is_empty(self):
        return len(self.items) == 0

q = Queue()
q.enqueue(1)
q.enqueue(2)
q.enqueue(3)
print(q.dequeue())  # 1
```

---

## 🎯 Casos

✅ **BFS (Breadth First Search)**  
✅ **Task Scheduling**  
✅ **Printer Queue**  
✅ **Process Scheduling**

---

## 🔗 Relacionado

- [[cs-stack|Stack]]
- [[cs-linked-list|Linked List]]

---

**Status:** ✅ Completo
