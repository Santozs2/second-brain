---
type: concept
id: cs-cpu-architecture
created: 2026-07-05
updated: 2026-07-05
category: Hardware
tags:
  - type/concept
  - domain/hardware
  - difficulty/intermediate
---

# 🖥️ CPU Architecture

> Projeto da Unidade Central de Processamento.

---

## 📖 Ciclo Fetch-Decode-Execute

```
1. FETCH:   Busca instrução da memória
2. DECODE:  Decodifica o que fazer
3. EXECUTE: Executa operação
4. STORE:   Armazena resultado
```

---

## 🏗️ Componentes

```
┌─────────────────────────────────┐
│ Control Unit (CU)               │
│ - Coordena operações            │
├─────────────────────────────────┤
│ Arithmetic Logic Unit (ALU)     │
│ - +, -, *, /, comparações       │
├─────────────────────────────────┤
│ Registradores                   │
│ - Armazenamento ultra-rápido    │
├─────────────────────────────────┤
│ Cache                           │
│ - L1, L2, L3                    │
└─────────────────────────────────┘
```

---

## 🎯 Arquiteturas

**Von Neumann:** 1 memória, instruções + dados  
**Harvard:** 2 memórias separadas (mais rápido)

---

## 📊 Pipelining

Múltiplas instruções em paralelo:

```
Ciclo 1: FETCH inst1
Ciclo 2: FETCH inst2, DECODE inst1
Ciclo 3: FETCH inst3, DECODE inst2, EXECUTE inst1
...
```

Aumenta throughput 3-4x

---

## 🔗 Relacionado

- [[cs-memory-hierarchy|Memory Hierarchy]]
- [[cs-alu-design|ALU Design]]

---

**Status:** ✅ Completo
