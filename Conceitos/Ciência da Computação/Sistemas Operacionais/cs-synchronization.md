---
type: concept
area: Conceitos
difficulty: advanced
id: cs-synchronization
category: Operating Systems
tags:
  - operating-systems
created: 2026-07-05
updated: 2026-07-05
---
# 🔒 Synchronization

> Coordenar múltiplos processos/threads.

---

## 🧠 Race Condition

```python
# Sem sincronização - bug!
counter = 0

def increment():
    global counter
    counter += 1  # Não é atômico!

# Thread 1: READ counter (5)
# Thread 2: READ counter (5)
# Thread 1: WRITE counter (6)
# Thread 2: WRITE counter (6)
# Resultado: 6 (deveria ser 7)
```

---

## 🔧 Primitivas

### Mutex (Mutual Exclusion)
```python
import threading

lock = threading.Lock()

with lock:
    # Seção crítica - apenas 1 thread
    counter += 1
```

### Semaphore
```
# Contador para N recursos
sem = Semaphore(3)  # 3 recursos
sem.acquire()       # -1
sem.release()       # +1
```

### Monitor
```python
# Combinação lock + condition variable
condition = threading.Condition(lock)
condition.wait()    # Espera
condition.notify()  # Acorda
```

---

## ⚠️ Deadlock

Condições necessárias:
1. Mutual Exclusion (recurso exclusivo)
2. Hold and Wait (segura + pede mais)
3. No Preemption (não pode tirar)
4. Circular Wait (ciclo de espera)

**Prevenção:** Remove 1+ condição

---

## 🎯 Exemplos

✅ **Banco:** Transações (all or nothing)  
✅ **Produtor-Consumidor:** Buffer compartilhado  
✅ **Readers-Writers:** Múltiplos leitores, 1 escritor

---

**Status:** ✅ Completo
