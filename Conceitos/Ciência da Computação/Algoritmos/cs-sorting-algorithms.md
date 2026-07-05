---
type: concept
id: cs-sort-algos
created: 2026-07-04
updated: 2026-07-04
category: Algorithms
tags:
  - type/concept
  - domain/algorithms
  - status/stable
  - difficulty/intermediate
---

# 🔄 Algoritmos de Ordenação

> Organizar elementos em ordem específica (crescente ou decrescente).

---

## 📊 Comparação

| Algoritmo | Tempo Médio | Pior Caso | Espaço | Estável | Uso |
|-----------|------------|----------|--------|---------|-----|
| **Bubble Sort** | O(n²) | O(n²) | O(1) | ✅ | Educacional |
| **Selection Sort** | O(n²) | O(n²) | O(1) | ❌ | Educacional |
| **Insertion Sort** | O(n²) | O(n²) | O(1) | ✅ | Pequenos arrays |
| **Merge Sort** | O(n log n) | O(n log n) | O(n) | ✅ | Dados não-RAM |
| **Quick Sort** | O(n log n) | O(n²) | O(log n) | ❌ | Uso geral |
| **Heap Sort** | O(n log n) | O(n log n) | O(1) | ❌ | Espaço limitado |
| **Counting Sort** | O(n+k) | O(n+k) | O(k) | ✅ | Inteiros |
| **Radix Sort** | O(nk) | O(nk) | O(n+k) | ✅ | Números grandes |

---

## 🟡 Bubble Sort

### Definição
Compara elementos adjacentes e troca se estiverem na ordem errada.

### Pseudocódigo
```
BUBBLE_SORT(array)
    n = length(array)
    para i = 0 até n-1:
        para j = 0 até n-i-2:
            se array[j] > array[j+1]:
                trocar(array[j], array[j+1])
```

### Python
```python
def bubble_sort(array):
    n = len(array)
    for i in range(n):
        swapped = False
        for j in range(0, n - i - 1):
            if array[j] > array[j + 1]:
                array[j], array[j + 1] = array[j + 1], array[j]
                swapped = True
        if not swapped:
            break
    return array

# Exemplo
numeros = [5, 2, 8, 1, 9]
print(bubble_sort(numeros))  # [1, 2, 5, 8, 9]
```

### Visualização
```
Pass 1: [5, 2, 8, 1, 9] → [2, 5, 1, 8, 9]  (9 em posição)
Pass 2: [2, 5, 1, 8, 9] → [2, 1, 5, 8, 9]  (8 em posição)
Pass 3: [2, 1, 5, 8, 9] → [1, 2, 5, 8, 9]  (5 em posição)
Pass 4: [1, 2, 5, 8, 9] → [1, 2, 5, 8, 9]  (pronto)
```

### Análise
- **Tempo:** O(n²) sempre
- **Espaço:** O(1)
- **Estável:** ✅ Sim
- **Uso:** Educacional APENAS

---

## 🟡 Insertion Sort

### Definição
Insere cada elemento na posição correta de um subarray já ordenado.

### Python
```python
def insertion_sort(array):
    for i in range(1, len(array)):
        chave = array[i]
        j = i - 1
        
        while j >= 0 and array[j] > chave:
            array[j + 1] = array[j]
            j -= 1
        
        array[j + 1] = chave
    
    return array

# Exemplo
numeros = [5, 2, 8, 1, 9]
print(insertion_sort(numeros))  # [1, 2, 5, 8, 9]
```

### Análise
- **Melhor caso:** O(n) - array já ordenado
- **Pior caso:** O(n²)
- **Espaço:** O(1)
- **Estável:** ✅ Sim
- **Uso:** Arrays pequenos, semi-ordenados

---

## 🟢 Merge Sort

### Definição
Divide o array em metades, ordena, e combina (Dividir & Conquistar).

### Pseudocódigo
```
MERGE_SORT(array, esquerda, direita)
    se esquerda < direita:
        meio = (esquerda + direita) // 2
        MERGE_SORT(array, esquerda, meio)
        MERGE_SORT(array, meio+1, direita)
        MERGE(array, esquerda, meio, direita)

MERGE(array, esquerda, meio, direita)
    // Combina dois subarrays ordenados
```

### Python
```python
def merge_sort(array):
    if len(array) <= 1:
        return array
    
    meio = len(array) // 2
    esquerda = merge_sort(array[:meio])
    direita = merge_sort(array[meio:])
    
    return merge(esquerda, direita)

def merge(esquerda, direita):
    resultado = []
    i = j = 0
    
    while i < len(esquerda) and j < len(direita):
        if esquerda[i] <= direita[j]:
            resultado.append(esquerda[i])
            i += 1
        else:
            resultado.append(direita[j])
            j += 1
    
    resultado.extend(esquerda[i:])
    resultado.extend(direita[j:])
    
    return resultado

# Exemplo
numeros = [5, 2, 8, 1, 9]
print(merge_sort(numeros))  # [1, 2, 5, 8, 9]
```

### Visualização
```
[5, 2, 8, 1, 9]
   ↓ Divide
[5, 2] [8, 1, 9]
  ↓       ↓
[5] [2] [8] [1, 9]
         ↓
       [8] [1] [9]
   ↓ Merge
[2, 5] [1, 8, 9]
   ↓
[1, 2, 5, 8, 9]
```

### Análise
- **Tempo:** O(n log n) SEMPRE
- **Espaço:** O(n) - precisa espaço extra
- **Estável:** ✅ Sim
- **Uso:** Grandes volumes, dados externos (disco)

---

## 🟢 Quick Sort

### Definição
Escolhe pivô, particiona, e ordena recursivamente (Dividir & Conquistar).

### Python
```python
def quick_sort(array):
    if len(array) <= 1:
        return array
    
    pivo = array[len(array) // 2]
    menores = [x for x in array if x < pivo]
    iguais = [x for x in array if x == pivo]
    maiores = [x for x in array if x > pivo]
    
    return quick_sort(menores) + iguais + quick_sort(maiores)

# Mais eficiente (in-place):
def quick_sort_inplace(array, inicio=0, fim=None):
    if fim is None:
        fim = len(array) - 1
    
    if inicio < fim:
        pivo_idx = particionar(array, inicio, fim)
        quick_sort_inplace(array, inicio, pivo_idx - 1)
        quick_sort_inplace(array, pivo_idx + 1, fim)
    
    return array

def particionar(array, inicio, fim):
    pivo = array[fim]
    i = inicio - 1
    
    for j in range(inicio, fim):
        if array[j] < pivo:
            i += 1
            array[i], array[j] = array[j], array[i]
    
    array[i + 1], array[fim] = array[fim], array[i + 1]
    return i + 1

# Exemplo
numeros = [5, 2, 8, 1, 9]
print(quick_sort(numeros))  # [1, 2, 5, 8, 9]
```

### Análise
- **Caso médio:** O(n log n)
- **Pior caso:** O(n²) - pivô ruim
- **Espaço:** O(log n) recursão
- **Estável:** ❌ Não
- **Uso:** Propósito geral, muito rápido na prática

---

## 🟢 Heap Sort

### Definição
Usa heap (árvore binária) para encontrar mínimo/máximo.

### Python
```python
def heap_sort(array):
    n = len(array)
    
    # Constrói max heap
    for i in range(n // 2 - 1, -1, -1):
        heapify(array, n, i)
    
    # Extrai elementos um a um
    for i in range(n - 1, 0, -1):
        array[0], array[i] = array[i], array[0]
        heapify(array, i, 0)
    
    return array

def heapify(array, n, i):
    maior = i
    esquerda = 2 * i + 1
    direita = 2 * i + 2
    
    if esquerda < n and array[esquerda] > array[maior]:
        maior = esquerda
    
    if direita < n and array[direita] > array[maior]:
        maior = direita
    
    if maior != i:
        array[i], array[maior] = array[maior], array[i]
        heapify(array, n, maior)

# Exemplo
numeros = [5, 2, 8, 1, 9]
print(heap_sort(numeros))  # [1, 2, 5, 8, 9]
```

### Análise
- **Tempo:** O(n log n) SEMPRE
- **Espaço:** O(1)
- **Estável:** ❌ Não
- **Uso:** Espaço limitado, tempo garantido

---

## 💡 Quando Usar Qual

```
┌─────────────────────────────────────┐
│ Escolha de Algoritmo                │
├─────────────────────────────────────┤
│                                     │
│ Array pequeno (< 50)               │
│ ↓                                   │
│ Insertion Sort (mais rápido)        │
│                                     │
│ Array grande, espaço ilimitado      │
│ ↓                                   │
│ Quick Sort (mais rápido em média)   │
│                                     │
│ Array grande, tempo garantido       │
│ ↓                                   │
│ Merge Sort ou Heap Sort             │
│                                     │
│ Espaço crítico                      │
│ ↓                                   │
│ Heap Sort                           │
│                                     │
│ Dados externos (disco)              │
│ ↓                                   │
│ Merge Sort                          │
│                                     │
└─────────────────────────────────────┘
```

---

## ❓ Perguntas de Entrevista

1. **Melhor ordenação geral?**
   - Quick Sort (rápido), Merge Sort (garantido)

2. **Bubble sort usado comercialmente?**
   - Nunca - O(n²) é inaceitável

3. **Qual gasta menos memória?**
   - Heap Sort (O(1) extra)

4. **Qual mantém ordem original de iguais?**
   - Estáveis: Bubble, Insertion, Merge

5. **Pior cenário Quick Sort?**
   - Array já ordenado + pivô ruim = O(n²)

---

## 📝 Exercícios

1. Implemente 3 algoritmos diferentes
2. Compare tempos reais para n=10,000
3. Qual é mais rápido?
4. Estude hybrid sort (Tim Sort)
5. Implemente Counting Sort

---

## 🔗 Referências Cruzadas

- [[cs-big-o|Big O Notation]]
- [[cs-algorithm-intro|Introdução a Algoritmos]]
- [[cs-search-algorithms|Algoritmos de Busca]]
- [[cs-array|Array]]
- [[cs-heap|Heap]]

---

**Próximo:** [[cs-graph-algorithms|Algoritmos em Grafos]]

**Status:** ✅ Completo  
**Dificuldade:** ⭐⭐ Intermediário  
**Tempo de Leitura:** 30-40 minutos
