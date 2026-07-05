---
type: concept
area: Conceitos
difficulty: advanced
id: cs-alu-design
category: Hardware
tags:
  - hardware
created: 2026-07-05
updated: 2026-07-05
---
# 🧮 Design da ALU

> A **Unidade Lógica e Aritmética** é o circuito da CPU que executa operações aritméticas e lógicas.

---

## 💡 O que faz

A ALU recebe dois operandos e um código de operação (opcode) e produz um resultado mais **flags** de status.

```
        A ──┐
            ├──► [ ALU ] ──► Resultado
        B ──┘      ▲
                   │
              opcode (seleciona operação)
```

---

## ⚙️ Operações típicas

- **Aritméticas**: soma, subtração, incremento, comparação
- **Lógicas**: AND, OR, XOR, NOT
- **Deslocamento**: shift left/right, rotação

Ver [[cs-boolean-logic|Lógica Booleana]] para as portas que a compõem.

---

## 🚩 Flags de Status

| Flag | Significa |
|------|-----------|
| Zero (Z) | Resultado é zero |
| Carry (C) | Vai-um / empréstimo |
| Overflow (V) | Estouro em sinalizado |
| Sign (S) | Resultado negativo |

Essas flags alimentam decisões de desvio condicional (`if`, loops).

---

## 🏗️ Construção

- **Somador completo (full adder)** é o bloco base: encadeados formam um ripple-carry adder.
- **Carry-lookahead** acelera a propagação do vai-um.
- Um **multiplexador** seleciona qual resultado (soma, AND, OR...) sai, conforme o opcode.

---

## 🔗 Relacionado

- [[cs-cpu-architecture|CPU Architecture]]
- [[cs-boolean-logic|Boolean Logic]]
- [[cs-memory-hierarchy|Memory Hierarchy]]

---

**Status:** ✅ Completo
