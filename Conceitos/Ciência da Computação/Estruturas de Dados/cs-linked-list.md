---
type: concept
id: cs-linked-list
created: 2026-07-04
updated: 2026-07-04
category: Data Structures
tags:
  - type/concept
  - domain/data-structures
  - difficulty/intermediate
---

# 🔗 Linked List

> Coleção dinâmica onde cada nó aponta para o próximo.

---

## 📖 Definição

Uma **Linked List** é:
- **Dinâmica:** Cresce/encolhe em runtime
- **Não-contígua:** Espalhada na memória
- **Nós:** Cada nó tem valor + referência
- **Tipos:** Simples, Dupla, Circular

---

## 🏗️ Estrutura

### Simples (Singly Linked List)
```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None
    
    def append(self, data):
        if not self.head:
            self.head = Node(data)
            return
        current = self.head
        while current.next:
            current = current.next
        current.next = Node(data)
    
    def display(self):
        current = self.head
        while current:
            print(current.data, end=" -> ")
            current = current.next
        print("None")

# Uso
ll = LinkedList()
ll.append(1)
ll.append(2)
ll.append(3)
ll.display()  # 1 -> 2 -> 3 -> None
```

### Dupla (Doubly Linked List)
```python
class DNode:
    def __init__(self, data):
        self.data = data
        self.next = None
        self.prev = None
```

### Circular
```
[1] -> [2] -> [3] -> [1] (volta ao inicio)
```

---

## ⏱️ Operações

| Operação | Tempo | Espaço |
|----------|-------|--------|
| Acesso | O(n) | O(1) |
| Busca | O(n) | O(1) |
| Inserção | O(n) | O(1) |
| Deleção | O(n) | O(1) |
| Início | O(1) | O(1) |

---

## 💡 Comparação com Array

| Aspecto | Array | Linked List |
|---------|-------|-------------|
| Acesso | O(1) ✅ | O(n) |
| Inserção | O(n) | O(1) ✅ |
| Deleção | O(n) | O(1) ✅ |
| Memória | Contígua | Espalhada |
| Cache | Melhor | Pior |

---

## 🎯 Casos de Uso

✅ **Undo/Redo**  
✅ **Fila (Queue)**  
✅ **Hash table com chaining**  
✅ **Música: playlist**

---

## ❓ Entrevista

1. **Detectar ciclo em linked list?**
   - Floyd's Cycle Detection (tortoise & hare)

2. **Reverse linked list?**
   - 3 pointers: prev, current, next

3. **Merge duas listas?**
   - Compare e concatene

4. **Remove duplicatas?**
   - Hash set ou two pointers

5. **Find middle?**
   - Slow & fast pointers

---

## 🔗 Relacionado

- [[cs-array|Array]]
- [[cs-stack|Stack]]
- [[cs-queue|Queue]]

---

**Status:** ✅ Completo  
**Dificuldade:** ⭐⭐ Intermediário
