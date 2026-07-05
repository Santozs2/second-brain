---
type: concept
area: Conceitos
difficulty: advanced
id: cs-dynamic-programming
category: Algorithms
tags:
  - algorithms
created: 2026-07-05
updated: 2026-07-05
---
# 🔄 Dynamic Programming

> Resolve problemas quebrando em subproblemas e reutilizando resultados.

---

## 📖 Definição

**DP** requer:
1. **Subestrutura Ótima:** Solução ótima = soluções ótimas dos subproblemas
2. **Subproblemas Sobrepostos:** Mesmos subproblemas resolvidos múltiplas vezes
3. **Memoização:** Cache dos resultados

---

## 💡 Abordagens

### Top-Down (Memoização)
```python
def fib_memo(n, memo={}):
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fib_memo(n-1, memo) + fib_memo(n-2, memo)
    return memo[n]

# Complexidade: O(n)
# Espaço: O(n)
```

### Bottom-Up (Tabulação)
```python
def fib_tab(n):
    if n <= 1:
        return n
    dp = [0] * (n + 1)
    dp[1] = 1
    
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    
    return dp[n]

# Complexidade: O(n)
# Espaço: O(n)
```

---

## 🎯 Exemplos Clássicos

### 1. Fibonacci
```
fib(5) = fib(4) + fib(3)
         /        \
    fib(3)+fib(2) fib(2)+fib(1)
    
Com memoização: O(n) instead of O(2^n)
```

### 2. 0/1 Knapsack
```python
def knapsack(weights, values, capacity):
    n = len(weights)
    dp = [[0] * (capacity + 1) for _ in range(n + 1)]
    
    for i in range(1, n + 1):
        for w in range(capacity + 1):
            # Não pegar item i
            dp[i][w] = dp[i-1][w]
            
            # Pegar item i (se couber)
            if weights[i-1] <= w:
                dp[i][w] = max(
                    dp[i][w],
                    values[i-1] + dp[i-1][w - weights[i-1]]
                )
    
    return dp[n][capacity]

# Complexidade: O(n*W) onde W = capacity
# Espaço: O(n*W)
```

### 3. Longest Common Subsequence (LCS)
```python
def lcs(text1, text2):
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    
    return dp[m][n]

# Uso: Diff, versionamento
```

### 4. Coin Change
```python
def coin_change(coins, amount):
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    
    for coin in coins:
        for x in range(coin, amount + 1):
            dp[x] = min(dp[x], dp[x - coin] + 1)
    
    return dp[amount] if dp[amount] != float('inf') else -1

# Complexidade: O(n*m) coins=m, amount=n
```

### 5. Edit Distance
```python
def editDistance(word1, word2):
    m, n = len(word1), len(word2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(m + 1):
        dp[i][0] = i
    for j in range(n + 1):
        dp[0][j] = j
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i-1] == word2[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(
                    dp[i-1][j],      # delete
                    dp[i][j-1],      # insert
                    dp[i-1][j-1]     # replace
                )
    
    return dp[m][n]

# Uso: Spell checker, fuzzy matching
```

---

## 📊 Padrões de DP

```
1D DP: dp[i] depende de dp[i-1], dp[i-2], etc.
2D DP: dp[i][j] depende de dp[i-1][j], dp[i][j-1]
String DP: Processar strings com DP
Interval DP: dp[i][j] = melhor solução para intervalo [i,j]
Tree DP: DP em árvores
```

---

## 🎯 Problemas Comuns

✅ **Fibonacci**  
✅ **Knapsack (0/1, unbounded)**  
✅ **LCS/LIS**  
✅ **Edit Distance**  
✅ **Coin Change**  
✅ **Matrix Chain Multiplication**  
✅ **Longest Increasing Subsequence**

---

## 💡 Dicas

1. **Identifique estado:** Qual informação precisa ser "lembrada"
2. **Defina transição:** Como passar de um estado para outro
3. **Base case:** Valores iniciais
4. **Otimização:** Reduction de espaço (2D → 1D)

---

## ❓ Entrevista

1. Implementar Fibonacci com memoização
2. 0/1 Knapsack
3. LCS ou Edit Distance
4. Longest Increasing Subsequence
5. Multiplicação de cadeias

---

## 🔗 Relacionado

- [[cs-divide-conquer|Divide & Conquer]]
- [[cs-greedy-algorithms|Greedy]]
- [[cs-recursion|Recursion]]

---

**Status:** ✅ Completo
