---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-tree
category: Data Structures
tags:
  - data-structures
created: 2026-07-05
updated: 2026-07-05
---
# 🌳 Tree (Árvore)

> Estrutura hierárquica com raiz, nós e folhas.

---

## 📖 Definição

**Tree** é:
- **Hierárquica:** Raiz (root) → Nós → Folhas
- **Acíclica:** Sem ciclos (diferente de grafo)
- **Conectada:** Caminho único entre quaisquer 2 nós
- **N nós = N-1 edges**
- **Altura:** Distância máxima raiz-folha

---

## 🧠 Estrutura

```python
class TreeNode:
    def __init__(self, data):
        self.data = data
        self.children = []

class BinaryTreeNode:
    def __init__(self, data):
        self.data = data
        self.left = None
        self.right = None
```

---

## 🎯 Tipos de Árvores

### Binary Tree
```
       1
      / \
     2   3
    / \
   4   5
```

### Binary Search Tree (BST)
```
       5
      / \
     3   7
    / \ / \
   1  4 6  8
```

Propriedade: Left < Parent < Right

### AVL Tree (Balanceado)
- Altura diferença ≤ 1
- Auto-balanceável com rotações
- Operações: O(log n)

### Red-Black Tree
- Nós pretos/vermelhos
- Propriedades de balanceamento
- Menos rotações que AVL

---

## 📊 Operações

| Operação | BST | AVL | Hash |
|----------|-----|-----|------|
| Insert | O(n) | O(log n) | O(1) |
| Delete | O(n) | O(log n) | O(1) |
| Search | O(n) | O(log n) | O(1) |

---

## 💻 Traversals

```python
# In-order (esquerda-raiz-direita)
# Resultado sorted em BST
def inorder(node):
    if node:
        inorder(node.left)
        print(node.data)
        inorder(node.right)

# Pre-order (raiz-esquerda-direita)
def preorder(node):
    if node:
        print(node.data)
        preorder(node.left)
        preorder(node.right)

# Post-order (esquerda-direita-raiz)
def postorder(node):
    if node:
        postorder(node.left)
        postorder(node.right)
        print(node.data)

# Level-order (BFS)
from collections import deque
def levelorder(root):
    queue = deque([root])
    while queue:
        node = queue.popleft()
        print(node.data)
        if node.left:
            queue.append(node.left)
        if node.right:
            queue.append(node.right)
```

---

## 🎯 Casos de Uso

✅ **Sistemas de Arquivos**  
✅ **Índices de Banco de Dados**  
✅ **Parsing (Compiladores)**  
✅ **Hierarquias (Org, DOM)**  
✅ **Busca binária**

---

## ❓ Entrevista

1. Implementar BST (insert, delete, search)
2. AVL tree rotações
3. Lowest Common Ancestor (LCA)
4. Serialize/Deserialize tree
5. Morris in-order traversal (sem recursão)

---

## 🔗 Relacionado

- [[cs-graph|Graph]]
- [[cs-heap|Heap]]
- [[cs-array|Array]]

---

**Status:** ✅ Completo
