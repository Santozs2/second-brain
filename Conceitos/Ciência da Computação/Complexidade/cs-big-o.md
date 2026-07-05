---
type: concept
area: Conceitos
status: stable
difficulty: beginner
id: cs-bigoo-notation
category: Complexity
tags:
  - complexity
created: 2026-07-04
updated: 2026-07-04
---
# 📈 Big O Notation

> Método para descrever como o tempo de execução ou espaço cresce com o tamanho da entrada.

---

## 📖 Definição

Big O descreve o **pior caso** - a máxima quantidade de operações.

```
O(f(n)) significa que o algoritmo faz no máximo f(n) operações
```

---

## 📊 Notações Comuns (do melhor ao pior)

```
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(n³) < O(2ⁿ) < O(n!)
```

### **O(1) - Constante**
```
Operações: Sempre as mesmas, independente de n
Exemplo: Acessar elemento de array por índice

def primeiro_elemento(array):
    return array[0]  # Sempre 1 operação
```

**Gráfico:**
```
|     
|─────────────────  (linha horizontal)
|
└─────────────────
  n
```

---

### **O(log n) - Logarítmico**
```
Operações: Crescem lentamente (dividindo por 2 cada vez)
Exemplo: Busca binária

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
```

**Gráfico:**
```
|     /
|    /
|   /─────────
|  /
| /
└────────────
  n
```

---

### **O(n) - Linear**
```
Operações: Crescem proporcionalmente a n
Exemplo: Busca linear, iterar array

def busca_linear(array, alvo):
    for elemento in array:
        if elemento == alvo:
            return True
    return False
```

**Gráfico:**
```
|      /
|     /
|    /
|   /
|  /
| /
└────────────
  n
```

---

### **O(n log n) - Linearítmico**
```
Operações: n × log(n)
Exemplo: Merge sort, Quick sort (médio)

def merge_sort(array):
    if len(array) <= 1:
        return array
    
    meio = len(array) // 2
    esquerda = merge_sort(array[:meio])
    direita = merge_sort(array[meio:])
    return merge(esquerda, direita)
```

**Gráfico:**
```
|       /
|      /
|     /
|    /─────
|   /
|  /
└────────────
  n
```

---

### **O(n²) - Quadrático**
```
Operações: n × n
Exemplo: Bubble sort, Insertion sort

def bubble_sort(array):
    n = len(array)
    for i in range(n):
        for j in range(n - 1 - i):
            if array[j] > array[j + 1]:
                array[j], array[j + 1] = array[j + 1], array[j]
    return array
```

**Gráfico:**
```
|       /
|      /
|     /
|    /
|   /
|  /
| /
└────────────
  n
```

---

### **O(2ⁿ) - Exponencial**
```
Operações: Duplicam a cada aumento de n
Exemplo: Fibonacci recursivo, brute force

def fibonacci_exponencial(n):
    if n <= 1:
        return n
    return fibonacci_exponencial(n-1) + fibonacci_exponencial(n-2)
```

**Gráfico:**
```
|                /
|               /
|              /
|             /
|            /
|           /
|          /
└────────────
  n
```

---

### **O(n!) - Fatorial**
```
Operações: n!
Exemplo: Todas as permutações

def todas_permutacoes(array):
    if len(array) <= 1:
        return [array]
    
    resultado = []
    for i, elemento in enumerate(array):
        resto = array[:i] + array[i+1:]
        for perm in todas_permutacoes(resto):
            resultado.append([elemento] + perm)
    return resultado
```

**Gráfico:**
```
|              /
|             /
|            /
|           /
|          /
|         /
|        /
|       /
└────────────
  n
```

---

## 🎯 Comparação Visual

```
n = 10:
O(1):    1
O(log n): 3
O(n):    10
O(n log n): 33
O(n²):   100
O(n³):   1000
O(2ⁿ):   1024
O(n!):   3.6 milhões

n = 1000:
O(1):    1
O(log n): 10
O(n):    1000
O(n log n): 10,000
O(n²):   1,000,000
O(n³):   1,000,000,000
O(2ⁿ):   IMPOSSÍVEL
O(n!):   IMPOSSÍVEL
```

---

## 🧮 Como Calcular Big O

### Regra 1: Ignore Constantes
```python
def exemplo(n):
    a = 5           # O(1)
    b = 10          # O(1)
    for i in range(n):      # O(n)
        print(i)    # O(1)

# Total: O(1) + O(1) + O(n) + O(1) = O(n)
# Ignoramos as constantes
```

### Regra 2: Pegue Termo Dominante
```python
def exemplo(n):
    for i in range(n):      # O(n)
        print(i)
    
    for i in range(n):      # O(n)
        for j in range(n):  # O(n²)
            print(i, j)

# Total: O(n) + O(n²) = O(n²)
# O(n²) domina, ignoramos O(n)
```

### Regra 3: Loops Aninhados = Multiplicação
```python
def exemplo(n, m):
    for i in range(n):      # O(n)
        for j in range(m):  # O(m)
            print(i, j)

# Total: O(n) × O(m) = O(n × m) ou O(n²) se n = m
```

---

## 💡 Dicas Práticas

| Estrutura | Complexidade |
|-----------|------------|
| Single loop | O(n) |
| Nested loops (2) | O(n²) |
| Nested loops (3) | O(n³) |
| Divide & conquer | O(log n) ou O(n log n) |
| Recursão simples | O(2ⁿ) |
| Fibonacci recursivo | O(φⁿ) ≈ O(1.618ⁿ) |

---

## ❓ Perguntas de Entrevista

1. **O que é Big O?**
   - Notação para descrever crescimento de tempo/espaço

2. **Por que O(n²) é pior que O(n log n)?**
   - Para n=1000: n² = 1M, n log n = 10K

3. **Binary search é O(log n). Por quê?**
   - Elimina metade dos elementos a cada passo

4. **Qual a diferença entre O(n) e O(2n)?**
   - Nenhuma! Ambas são O(n) (ignora constantes)

5. **Bubble sort vs Merge sort?**
   - Bubble: O(n²), Merge: O(n log n)

---

## 📝 Exercícios

1. Calcule Big O para busca linear
2. Calcule Big O para busca binária
3. Qual é mais eficiente?
4. Escreva código que é O(n log n)
5. Escreva código que é O(2ⁿ) e veja quando fica impossível

---

## 🔗 Referências Cruzadas

- [[cs-algorithm-intro|Introdução a Algoritmos]]
- [[cs-search-algorithms|Algoritmos de Busca]]
- [[cs-sorting-algorithms|Algoritmos de Ordenação]]
- [[cs-asymptotic-analysis|Análise Assintótica]]
- [[cs-array|Array]]

---

**Próximo:** [[cs-search-algorithms|Algoritmos de Busca]]

**Status:** ✅ Completo  
**Dificuldade:** ⭐⭐ Intermediário  
**Tempo de Leitura:** 20-30 minutos
