---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-probability
category: Mathematics
tags:
  - mathematics
created: 2026-07-05
updated: 2026-07-05
---
# 🎲 Probability

> Teoria de eventos aleatórios.

---

## 📖 Conceitos

```
P(A) = Número de casos favoráveis / Total de casos

P(A) = 0      → Impossível
P(A) = 0.5    → 50% chance
P(A) = 1      → Certo
```

---

## 🧮 Regras

```python
# Complemento
P(A') = 1 - P(A)

# União (A ou B)
P(A ∪ B) = P(A) + P(B) - P(A ∩ B)

# Interseção (A e B)
P(A ∩ B) = P(A) * P(B|A)

# Condicional
P(A|B) = P(A ∩ B) / P(B)

# Bayes
P(A|B) = P(B|A) * P(A) / P(B)
```

---

## 🎯 Distribuições

- **Binomial:** n tentativas, p sucesso
- **Normal:** Distribuição em sino
- **Poisson:** Eventos em tempo

---

## 🎯 Aplicações

✅ **Machine Learning**  
✅ **Estatística**  
✅ **Análise de risco**

---

**Status:** ✅ Completo
