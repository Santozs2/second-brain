---
type: concept
id: cs-graph-algorithms
created: 2026-07-05
updated: 2026-07-05
category: Algorithms
tags:
  - type/concept
  - domain/algorithms
  - difficulty/advanced
---

# 🔍 Graph Algorithms

> Algoritmos para busca, caminho mais curto, e mais.

---

## 1️⃣ BFS (Breadth First Search)

**Busca em largura - camada por camada**

```python
from collections import deque

def bfs(graph, start):
    visited = set()
    queue = deque([start])
    visited.add(start)
    
    while queue:
        vertex = queue.popleft()
        print(vertex)
        
        for neighbor in graph[vertex]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

# Complexidade: O(V + E)
# Espaço: O(V)
# Uso: Caminho mais curto em grafo não-ponderado
```

---

## 2️⃣ DFS (Depth First Search)

**Busca em profundidade - vai fundo antes de voltar**

```python
def dfs(graph, start, visited=None):
    if visited is None:
        visited = set()
    
    visited.add(start)
    print(start)
    
    for neighbor in graph[start]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)

# Iterativo com Stack:
def dfs_iterative(graph, start):
    visited = set()
    stack = [start]
    
    while stack:
        vertex = stack.pop()
        if vertex not in visited:
            visited.add(vertex)
            print(vertex)
            stack.extend(reversed(graph[vertex]))

# Complexidade: O(V + E)
# Espaço: O(V) recursão stack
# Uso: Detecção de ciclo, topological sort
```

---

## 3️⃣ Dijkstra (Caminho Mais Curto)

**Encontra caminho mais curto de um nó a todos outros**

```python
import heapq

def dijkstra(graph, start):
    distances = {node: float('inf') for node in graph}
    distances[start] = 0
    pq = [(0, start)]  # (distância, nó)
    visited = set()
    
    while pq:
        current_dist, current = heapq.heappop(pq)
        
        if current in visited:
            continue
        
        visited.add(current)
        
        for neighbor, weight in graph[current]:
            distance = current_dist + weight
            
            if distance < distances[neighbor]:
                distances[neighbor] = distance
                heapq.heappush(pq, (distance, neighbor))
    
    return distances

# Complexidade: O((V + E) log V) com heap
# Pré-requisito: Sem arestas negativas
# Uso: GPS, roteamento de rede
```

---

## 4️⃣ Floyd-Warshall (Todos para Todos)

**Encontra caminho mais curto entre TODOS pares**

```python
def floyd_warshall(n, edges):
    # Inicializar matrix
    dist = [[float('inf')] * n for _ in range(n)]
    
    # Distância de si mesmo = 0
    for i in range(n):
        dist[i][i] = 0
    
    # Adicionar arestas
    for u, v, w in edges:
        dist[u][v] = w
    
    # DP com 3 loops
    for k in range(n):
        for i in range(n):
            for j in range(n):
                dist[i][j] = min(
                    dist[i][j],
                    dist[i][k] + dist[k][j]
                )
    
    return dist

# Complexidade: O(V³)
# Funciona com arestas negativas (sem ciclos negativos)
# Uso: Matriz de caminhos
```

---

## 5️⃣ Topological Sort

**Ordena nós de DAG respeitando dependências**

```python
def topological_sort(graph):
    visited = set()
    stack = []
    
    def dfs(node):
        visited.add(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                dfs(neighbor)
        stack.append(node)
    
    for node in graph:
        if node not in visited:
            dfs(node)
    
    return stack[::-1]

# Uso: Task scheduling, resolvedor de dependências
```

---

## 📊 Comparação

| Algoritmo | Tempo | Espaço | Negativos |
|-----------|-------|--------|-----------|
| BFS | O(V+E) | O(V) | Sim (não-ponderado) |
| DFS | O(V+E) | O(V) | Sim |
| Dijkstra | O((V+E)logV) | O(V) | Não |
| Floyd-W | O(V³) | O(V²) | Sim |

---

## 🎯 Problemas Clássicos

✅ **Número de Ilhas:** DFS/BFS  
✅ **Caminho Existente?:** BFS  
✅ **Distância Mínima:** Dijkstra/BFS  
✅ **Ciclo Existente?:** DFS  
✅ **Ordem de Tasks:** Topological Sort

---

## ❓ Entrevista

1. Implementar BFS e DFS
2. Detecção de ciclo (direcionado/não-direcionado)
3. Número de componentes conexas
4. Dijkstra vs Bellman-Ford
5. Topological sort com Kahn's algorithm

---

## 🔗 Relacionado

- [[cs-graph|Graph]]
- [[cs-tree|Tree]]
- [[cs-dynamic-programming|Dynamic Programming]]

---

**Status:** ✅ Completo
