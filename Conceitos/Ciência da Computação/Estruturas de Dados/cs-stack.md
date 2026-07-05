---
type: concept
area: Conceitos
difficulty: beginner
id: cs-stack
category: Data Structures
tags:
  - data-structures
created: 2026-07-04
updated: 2026-07-04
---
# 📚 Stack (Pilha)

> LIFO - Last In First Out. Como pilhar pratos.

---

## 📖 Definição

**Stack** é uma estrutura LIFO:
- **Push:** Adiciona no topo
- **Pop:** Remove do topo
- **Peek:** Vê o topo sem remover
- **Tempo:** O(1) para todas operações

---

## 💻 Implementação

```python
class Stack:
    def __init__(self):
        self.items = []
    
    def push(self, item):
        self.items.append(item)
    
    def pop(self):
        return self.items.pop() if not self.is_empty() else None
    
    def peek(self):
        return self.items[-1] if not self.is_empty() else None
    
    def is_empty(self):
        return len(self.items) == 0
    
    def size(self):
        return len(self.items)

# Uso
s = Stack()
s.push(1)
s.push(2)
s.push(3)
print(s.pop())  # 3
print(s.peek())  # 2
```

---

## 🎯 Aplicações

1. **Function Call Stack** - Recursão
2. **Undo/Redo** - Editores
3. **Expressão balanceada** - Parênteses
4. **DFS (Depth First Search)**
5. **Browser History** - Voltar

---

## ❓ Entrevista

1. Implementar stack com array
2. Detectar parênteses balanceados
3. Reverse string
4. Implementar Min Stack (mín em O(1))
5. Avaliar expressão pós-fixa

---

## 🔗 Relacionado

- [[cs-queue|Queue]]
- [[cs-linked-list|Linked List]]

---

**Status:** ✅ Completo
