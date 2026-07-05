---
type: concept
id: cs-search-algos
created: 2026-07-04
updated: 2026-07-04
category: Algorithms
tags:
  - type/concept
  - domain/algorithms
  - status/stable
  - difficulty/intermediate
---

# 🔍 Algoritmos de Busca

> Encontrar um elemento específico em uma coleção de dados.

---

## 📊 Comparação Rápida

| Algoritmo | Tempo | Espaço | Pré-requisito | Uso |
|-----------|-------|--------|--------------|-----|
| **Linear Search** | O(n) | O(1) | Nenhum | Array desordenado |
| **Binary Search** | O(log n) | O(1) | Array ordenado | Dados grandes |
| **Jump Search** | O(√n) | O(1) | Array ordenado | Meio termo |
| **Interpolation Search** | O(log log n) | O(1) | Dados uniformes | Distribuição regular |

---

## 🟡 Linear Search (Busca Sequencial)

### Definição
Verifica cada elemento sequencialmente até encontrar ou chegar ao fim.

### Pseudocódigo
```
BUSCA_LINEAR(array, alvo)
    para i = 0 até length(array) - 1:
        se array[i] == alvo:
            retorna i
    retorna -1
```

### Python
```python
def busca_linear(array, alvo):
    for i in range(len(array)):
        if array[i] == alvo:
            return i
    return -1

# Exemplo
numeros = [5, 2, 8, 1, 9]
print(busca_linear(numeros, 8))  # Output: 2
```

### Análise
- **Melhor caso:** O(1) - 1º elemento
- **Pior caso:** O(n) - último ou não existe
- **Caso médio:** O(n/2) ≈ O(n)
- **Espaço:** O(1)

### Quando Usar
✅ Array pequeno  
✅ Array desordenado  
✅ Precisa de primeira ocorrência  
❌ Array grande ordenado

---

## 🟢 Binary Search (Busca Binária)

### Definição
Divide o array em metades, eliminando metade a cada comparação.

### Pseudocódigo
```
BUSCA_BINARIA(array, alvo)
    esquerda = 0
    direita = length(array) - 1
    
    enquanto esquerda <= direita:
        meio = (esquerda + direita) // 2
        
        se array[meio] == alvo:
            retorna meio
        senão se array[meio] < alvo:
            esquerda = meio + 1
        senão:
            direita = meio - 1
    
    retorna -1
```

### Python
```python
def busca_binaria(array, alvo):
    esquerda, direita = 0, len(array) - 1
    
    while esquerda <= direita:
        meio = (esquerda + direita) // 2
        
        if array[meio] == alvo:
            return meio
        elif array[meio] < alvo:
            esquerda = meio + 1
        else:
            direita = meio - 1
    
    return -1

# Exemplo
numeros = [1, 2, 3, 5, 8, 13, 21, 34]
print(busca_binaria(numeros, 13))  # Output: 5
```

### Visualização
```
Array: [1, 2, 3, 5, 8, 13, 21, 34]
Alvo: 13

Passo 1: meio = 5 → array[5] = 13
        ┌─────────────────┐
        ↓                 ↓
       [1, 2, 3, 5, 8, 13, 21, 34]
                         ↑
                      ENCONTRADO!

Total de operações: 1 (melhor caso)
```

```
Array: [1, 2, 3, 5, 8, 13, 21, 34]
Alvo: 21

Passo 1: meio = 5 → array[5] = 13 < 21
        Procura à direita
        
Passo 2: meio = 6 → array[6] = 21 ✓
        ENCONTRADO!

Total de operações: 2
```

### Análise
- **Melhor caso:** O(1) - elemento no meio
- **Pior caso:** O(log n) - log₂(n)
- **Caso médio:** O(log n)
- **Espaço:** O(1)

### Comparação com Linear

```
n = 1,000,000 elementos

Linear Search: até 1,000,000 comparações
Binary Search: até log₂(1,000,000) ≈ 20 comparações!

Diferença: 50,000× MAIS RÁPIDO!
```

### Quando Usar
✅ Array ordenado  
✅ Array grande  
✅ Múltiplas buscas  
❌ Array desordenado (precisa ordenar primeiro)

---

## 🟠 Jump Search

### Ideia
Pula de k em k elementos, depois busca linearmente.

### Python
```python
import math

def jump_search(array, alvo):
    n = len(array)
    step = int(math.sqrt(n))
    prev = 0
    
    # Encontra o bloco onde alvo pode estar
    while array[min(step, n) - 1] < alvo:
        prev = step
        step += int(math.sqrt(n))
        if prev >= n:
            return -1
    
    # Busca linear no bloco
    while array[prev] < alvo:
        prev += 1
        if prev == min(step, n):
            return -1
    
    if array[prev] == alvo:
        return prev
    
    return -1
```

### Análise
- **Complexidade:** O(√n)
- **Espaço:** O(1)
- **Uso:** Dados uniformes, sem acesso aleatório

---

## 🟣 Interpolation Search

### Ideia
Estima posição baseado na distribuição dos dados.

### Python
```python
def interpolation_search(array, alvo):
    esquerda, direita = 0, len(array) - 1
    
    while (esquerda <= direita and 
           alvo >= array[esquerda] and 
           alvo <= array[direita]):
        
        # Estima a posição
        pos = (esquerda + 
               (direita - esquerda) * 
               (alvo - array[esquerda]) // 
               (array[direita] - array[esquerda]))
        
        if array[pos] == alvo:
            return pos
        elif array[pos] < alvo:
            esquerda = pos + 1
        else:
            direita = pos - 1
    
    return -1
```

### Análise
- **Melhor caso:** O(1)
- **Pior caso:** O(n)
- **Caso médio:** O(log log n) - muito rápido!
- **Uso:** Dados uniformes (IPs, timestamps, etc)

---

## 💡 Casos Reais

### 1. Google Search
- Bilhões de documentos indexados
- Binary search em índices
- + Ranking algoritmo

### 2. Database Queries
```sql
SELECT * FROM users WHERE id = 42;
```
- Usa B-Tree (variação de binary search)
- Indexação para rápido acesso

### 3. Autocompletar
```
Digite: "gat"
Resultados: gate, gateway, gather
```
- Busca prefixo em Trie
- Algoritmo de busca especializado

### 4. Biblioteca
```
Procurando livro por ISBN
ISBN: 978-0-13-468599-1
```
- Array ordenado por ISBN
- Binary search muito eficiente

---

## ❓ Perguntas de Entrevista

1. **Qual complexidade de binary search? Por quê?**
   - O(log n) - elimina metade a cada passo

2. **Binary search funciona em array desordenado?**
   - Não - precisa estar ORDENADO

3. **Linear vs Binary para 10 elementos?**
   - Ambas rápidas, mas binary é O(4), linear O(10)

4. **Como encontrar primeiro elemento >= alvo?**
   - Binary search com lógica modificada

5. **Jump search vs binary search?**
   - Binary: O(log n), Jump: O(√n)
   - Binary é melhor em geral

---

## 📝 Exercícios

### Básico
1. Implemente linear search
2. Implemente binary search
3. Compare tempos reais para n=10,000
4. Qual é mais rápido?

### Intermediário
5. Binary search de first/last occurrence
6. Binary search para encontrar piso/teto
7. Jump search vs binary search

### Avançado
8. Interpolation search para dados reais
9. Busca em array 2D
10. Busca em lista ligada (que tipo não pode usar?)

---

## 🔗 Referências Cruzadas

- [[cs-big-o|Big O Notation]]
- [[cs-algorithm-intro|Introdução a Algoritmos]]
- [[cs-sorting-algorithms|Algoritmos de Ordenação]]
- [[cs-array|Array]]
- [[cs-tree|Árvore]]

---

**Próximo:** [[cs-sorting-algorithms|Algoritmos de Ordenação]]

**Status:** ✅ Completo  
**Dificuldade:** ⭐⭐ Intermediário  
**Tempo de Leitura:** 25-35 minutos
