---
type: tech
area: Estudos
id: lang-py-syntax
category: Python
tags:
  - backend
  - python
created: 2026-07-05
updated: 2026-07-05
---
# 🐍 Python - Syntax & Types

---

## 📖 Tipos

```python
# Primitivos
type(42)           # <class 'int'>
type(3.14)         # <class 'float'>
type("hello")      # <class 'str'>
type(True)         # <class 'bool'>

# Collections
type([1,2,3])      # <class 'list'>
type((1,2,3))      # <class 'tuple'>
type({1,2,3})      # <class 'set'>
type({'k':'v'})    # <class 'dict'>
```

---

## 📝 Strings

```python
# F-strings
name = "Alice"
f"Hello {name}"    # "Hello Alice"

# Multiline
text = """Line 1
Line 2"""

# Methods
"HELLO".lower()    # "hello"
"hello".upper()    # "HELLO"
```

---

## 📦 Collections

```python
# List
lst = [1, 2, 3]
lst.append(4)      # [1,2,3,4]
lst[0]             # 1

# Tuple (imutável)
tpl = (1, 2, 3)
tpl[0]             # 1

# Dictionary
d = {'name': 'Alice'}
d['name']          # 'Alice'

# Set
s = {1, 2, 3}
2 in s             # True
```

---

## 🔄 List Comprehension

```python
[x*2 for x in range(5)]           # [0,2,4,6,8]
[x for x in range(10) if x%2==0]  # [0,2,4,6,8]
{x: x**2 for x in range(5)}       # {0:0, 1:1, 2:4, 3:9, 4:16}
```

---

**Status:** ✅ Completo
