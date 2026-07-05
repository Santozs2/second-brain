---
type: concept
id: cs-graph
created: 2026-07-05
updated: 2026-07-05
category: Data Structures
tags:
  - type/concept
  - domain/data-structures
  - difficulty/intermediate
---

# 🕸️ Graph (Grafo)

> Conjunto de nós (vértices) conectados por arestas.

---

## 📖 Definição

**Graph G = (V, E)**:
- **V:** Vértices (nós)
- **E:** Arestas (conexões)
- **Direcionado/Não-direcionado**
- **Ponderado/Não-ponderado**
- **Cíclico/Acíclico**

---

## 🧠 Representações

### 1. Adjacency List (mais eficiente)
```python
graph = {
    'A': ['B', 'C'],
    'B': ['A', 'D'],
    'C': ['A'],
    'D': ['B']
}
# Espaço: O(V + E)
# Acesso: O(1) verificar conectividade, O(E) iterar
```

### 2. Adjacency Matrix
```python
# Matriz V x V
matrix = [
    [0, 1, 1, 0],  # A
    [1, 0, 0, 1],  # B
    [1, 0, 0, 0],  # C
    [0, 1, 0, 0]   # D
]
# Espaço: O(V²)
# Acesso: O(1) para verificar
```

---

## 💻 Classe Graph

```python
class Graph:
    def __init__(self):
        self.graph = {}
    
    def add_vertex(self, v):
        if v not in self.graph:
            self.graph[v] = []
    
    def add_edge(self, u, v, weight=1):
        if u not in self.graph:
            self.add_vertex(u)
        if v not in self.graph:
            self.add_vertex(v)
        self.graph[u].append((v, weight))
        # Não-direcionado:
        # self.graph[v].append((u, weight))
    
    def get_neighbors(self, v):
        return self.graph.get(v, [])
```

---

## 🎯 Tipos de Grafos

```
Simples Não-direcionado:
    A --- B
    |     |
    C --- D

Direcionado:
    A → B ← C
    ↓     ↑
    D → E

Ponderado:
    A --5-- B
    |    \  |
    3     2 4
    |      \|
    C --1-- D

DAG (Directed Acyclic Graph):
Sem ciclos, usado em scheduling
```

---

## 📊 Propriedades

```python
# Grau (número de arestas)
# Entrada (direcionado)
# Saída (direcionado)

# Caminho: Sequência de vértices
# Ciclo: Caminho que volta ao início
# Conexo: Existe caminho entre quaisquer 2 vértices
# Árbol: Grafo conectado acíclico
```

---

## 🎯 Casos de Uso

✅ **Redes Sociais**  
✅ **Mapas (rotas)**  
✅ **Recomendação**  
✅ **Parsing (compiladores)**  
✅ **Planejamento**

---

## ❓ Entrevista

1. Implementar grafo com adjacency list
2. BFS e DFS
3. Detecção de ciclo
4. Componentes conexas
5. Topological Sort

---

## 🔗 Relacionado

- [[cs-graph-bfs|BFS]]
- [[cs-graph-dfs|DFS]]
- [[cs-dijkstra|Dijkstra]]
- [[cs-tree|Tree]]

---

**Status:** ✅ Completo
