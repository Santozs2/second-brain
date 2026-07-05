---
type: concept
area: Conceitos
difficulty: beginner
id: cs-set-theory
category: Mathematics
tags:
  - mathematics
created: 2026-07-05
updated: 2026-07-05
---
# 🔢 Set Theory

> Matemática de coleções de elementos.

---

## 📖 Definição

**Conjunto:** Coleção desordenada de elementos únicos

```
A = {1, 2, 3}
B = {2, 3, 4}
```

---

## 🔧 Operações

```python
# União: A ∪ B = {1,2,3,4}
A | B

# Interseção: A ∩ B = {2,3}
A & B

# Diferença: A - B = {1}
A - B

# Complemento: A' = todos exceto A
A.symmetric_difference(B)

# Subconjunto: A ⊆ B?
A.issubset(B)

# Disjunto: sem elementos em comum
A.isdisjoint(B)
```

---

## 📊 Propriedades

```
Comutativa:     A ∪ B = B ∪ A
Associativa:    (A ∪ B) ∪ C = A ∪ (B ∪ C)
Distributiva:   A ∪ (B ∩ C) = (A ∪ B) ∩ (A ∪ C)
De Morgan:      (A ∪ B)' = A' ∩ B'
```

---

## 🎯 Aplicações

✅ **Lógica booleana**  
✅ **Banco de dados (queries)**  
✅ **Tipo de dados (Set em Python)**

---

**Status:** ✅ Completo
