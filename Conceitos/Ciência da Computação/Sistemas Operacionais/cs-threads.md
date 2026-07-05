---
type: concept
id: cs-threads
created: 2026-07-05
updated: 2026-07-05
category: Operating Systems
tags:
  - type/concept
  - domain/operating-systems
  - difficulty/intermediate
---

# 🧵 Threads

> Múltiplas linhas de execução dentro de um processo.

---

## 📊 Threads vs Processes

```
Process:
  ├── Espaço de memória isolado
  ├── Heap separado
  ├── Stack separado
  └── Context switch caro

Thread:
  ├── Compartilham código + dados
  ├── Próprio stack
  ├── Próprios registros
  └── Context switch rápido
```

---

## 💻 Python Threads

```python
import threading

def task(name):
    print(f"Task {name}")

# Criar threads
t1 = threading.Thread(target=task, args=("A",))
t2 = threading.Thread(target=task, args=("B",))

t1.start()
t2.start()

t1.join()
t2.join()

print("Done")
```

---

## ⚠️ GIL (Global Interpreter Lock)

Python (CPython) tem GIL que previne execução paralela real de bytecode  
**Workarounds:** multiprocessing, asyncio, PyPy/Jython

---

## 🎯 Tipos

- **User-level threads:** No espaço do usuário
- **Kernel-level threads:** SO gerencia
- **Hybrid:** Combinação

---

## 🔗 Relacionado

- [[cs-process|Processes]]
- [[cs-synchronization|Synchronization]]

---

**Status:** ✅ Completo
