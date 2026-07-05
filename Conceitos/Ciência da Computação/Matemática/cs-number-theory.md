---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-number-theory
category: Mathematics
tags:
  - mathematics
created: 2026-07-05
updated: 2026-07-05
---
# 🔢 Number Theory

> Propriedades dos números inteiros.

---

## 🔑 Conceitos Principais

### GCD & LCM
```python
def gcd(a, b):
    while b:
        a, b = b, a % b
    return a

def lcm(a, b):
    return (a * b) // gcd(a, b)
```

### Números Primos
```python
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True

# Sieve of Eratosthenes (todos primos até n)
def sieve(n):
    is_p = [True] * (n + 1)
    is_p[0] = is_p[1] = False
    
    for i in range(2, int(n**0.5) + 1):
        if is_p[i]:
            for j in range(i*i, n + 1, i):
                is_p[j] = False
    
    return [i for i in range(n + 1) if is_p[i]]
```

### Modular Arithmetic
```python
# (a + b) % m = ((a % m) + (b % m)) % m
# (a * b) % m = ((a % m) * (b % m)) % m

# Modular exponentiation (rápido)
def pow_mod(base, exp, mod):
    result = 1
    base = base % mod
    while exp > 0:
        if exp % 2 == 1:
            result = (result * base) % mod
        exp = exp // 2
        base = (base * base) % mod
    return result
```

---

## 🎯 Aplicações

✅ **Criptografia (RSA)**  
✅ **Hashing**  
✅ **Checksum**  
✅ **Teoria de códigos**

---

**Status:** ✅ Completo

## 🔗 Ver também nesta área

- [[cs-boolean-logic]]
- [[cs-combinatorics]]
- [[cs-linear-algebra]]
