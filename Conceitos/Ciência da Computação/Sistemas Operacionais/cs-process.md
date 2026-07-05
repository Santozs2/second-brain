---
type: concept
area: Conceitos
difficulty: intermediate
id: cs-process
category: Operating Systems
tags:
  - operating-systems
created: 2026-07-05
updated: 2026-07-05
---
# ⚙️ Process

> Instância de programa em execução com seu próprio espaço de memória.

---

## 🧠 Estrutura

```
Process Control Block (PCB):
├── PID (Process ID)
├── State (New, Ready, Running, Blocked, Terminated)
├── Program Counter
├── Registers
├── Memory info
└── I/O info

Memory Layout:
┌──────────────┐
│ Stack ↓      │
├──────────────┤
│ Heap ↑       │
├──────────────┤
│ Data (BSS)   │
├──────────────┤
│ Text (Code)  │
└──────────────┘
```

---

## 📊 States

```
       fork()
New ────────→ Ready ──→ Running ──→ Terminated
                ↑        ↓
                └─ Blocked
```

---

## 💻 Operações

```python
# fork() - cria processo filho
# exec() - executa novo programa
# wait() - espera termo filho
# exit() - encerra processo

# C/Unix:
pid_t pid = fork();
if (pid == 0) {  // Filho
    exec...
} else {  // Pai
    wait(pid);
}
```

---

## 🎯 Context Switch

CPU alterna entre processos salvando/restaurando PCB

---

## 🔗 Relacionado

- [[cs-threads|Threads]]
- [[cs-scheduling|Process Scheduling]]

---

**Status:** ✅ Completo
