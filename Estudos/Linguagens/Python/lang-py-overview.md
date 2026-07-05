---
type: tech
area: Estudos
status: stable
difficulty: beginner
id: lang-py-overview
category: Python
tags:
  - backend
  - data-science
  - python
created: 2026-07-04
updated: 2026-07-04
---
# 🐍 Python - Overview & History

> The Most Popular Language - Simple, Powerful, Ubiquitous

---

## 📖 Definição

**Python** é:
- **Interpretada** - Executada linha por linha
- **Dinâmica** - Tipagem dinâmica
- **Legível** - Sintaxe clara e intuitiva
- **Multipropósito** - Web, Data, AI/ML, Scripting
- **Comunidade** - Enorme ecossistema (PyPI: 4.5M packages)

---

## 📜 História

### **1989 - Criação (Guido van Rossum)**
```
Dezembro 1989: Projeto começou
Filosofia: "Beautiful is better than ugly"
Nome: Monty Python (série de comédia britânica)
```

### **Versões Principais**

#### Python 1.0 (1994)
- Funcionalidades básicas
- Padrão estável

#### Python 2.x (2000-2010)
```python
# Print era statement
print "Hello World"

# Integer division arredonda para baixo
5 / 2 = 2  # Esperado: 2.5
```

#### **Python 3.0 - Grande Quebra (2008)** 🔥
```python
# Print é função
print("Hello World")

# Unicode por padrão
# True division
5 / 2 = 2.5  # Correto!

# Removeu map(), filter() retorna iterators
# Dict.keys() retorna view, não lista
```

#### Python 3.5+ (2015+)
```python
# Async/Await
async def fetch():
    result = await get_data()

# Type hints (opcional)
def greet(name: str) -> str:
    return f"Hello {name}"

# Match/Case (3.10+)
match value:
    case 1:
        print("Um")
    case 2:
        print("Dois")
```

---

## 🌍 Onde Python Roda

### **Desktop & CLI**
```python
# CLI simples
import sys
print(sys.argv)

# Desktop GUI (Tkinter, PyQt)
import tkinter as tk
```

### **Web (Backend)**
```python
# Django
from django.http import HttpResponse

def hello(request):
    return HttpResponse("Hello World")

# Flask
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello World'

# FastAPI (moderno, rápido)
from fastapi import FastAPI
app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello"}
```

### **Data Science & AI**
```python
# NumPy para arrays numéricos
import numpy as np
arr = np.array([1, 2, 3])

# Pandas para análise de dados
import pandas as pd
df = pd.read_csv('data.csv')

# Scikit-learn para ML
from sklearn.ensemble import RandomForestClassifier

# TensorFlow para Deep Learning
import tensorflow as tf
model = tf.keras.Sequential([...])

# PyTorch alternativa
import torch
```

### **Scripting & Automação**
```python
# Automação de tarefas
import os
import shutil

# Web scraping
import requests
from bs4 import BeautifulSoup

# Cron jobs Python
# Task scheduling
```

---

## 🎯 Paradigmas

### 1. **Imperativo (Procedural)**
```python
def calculate():
    total = 0
    for i in range(10):
        total += i
    return total
```

### 2. **Orientado a Objetos**
```python
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        print(f"{self.name} makes a sound")

class Dog(Animal):
    def speak(self):
        print(f"{self.name} barks")

dog = Dog("Rex")
dog.speak()  # Rex barks
```

### 3. **Funcional**
```python
# List comprehension
squares = [x**2 for x in range(10)]

# Filter, map, reduce
numbers = [1, 2, 3, 4, 5]
evens = filter(lambda x: x % 2 == 0, numbers)
doubled = map(lambda x: x * 2, evens)

# Functional style
from functools import reduce
result = reduce(lambda x, y: x + y, [1, 2, 3, 4])  # 10
```

---

## 💪 Strengths

✅ **Readabilidade** - Código limpo e legível  
✅ **Comunidade** - Enorme, ativa, acolhedora  
✅ **Bibliotecas** - Milhões de packages (PyPI)  
✅ **Data Science** - Padrão ouro (NumPy, Pandas, Sklearn)  
✅ **AI/ML** - TensorFlow, PyTorch dominantes  
✅ **Curva aprendizado** - Muito fácil de começar  
✅ **Prototipagem** - Rápida e iterativa  

---

## ⚠️ Weaknesses

❌ **Performance** - Mais lenta que C/C++/Go  
❌ **GIL (Global Interpreter Lock)** - Limita paralelismo true  
❌ **Memory** - Overhead maior  
❌ **Mobile** - Não nativo (apenas via compilação)  
❌ **Type Safety** - Dinâmica = erros em runtime  

---

## 📊 Comparação

| Aspecto | Python | Go | Rust | JavaScript |
|---------|--------|-----|------|-----------|
| **Legibilidade** | 10 | 8 | 6 | 7 |
| **Performance** | 5 | 8 | 10 | 6 |
| **Comunidade** | 10 | 7 | 6 | 10 |
| **Data Science** | 10 | 4 | 3 | 3 |
| **Web Backend** | 8 | 9 | 7 | 8 |

---

## 📈 Estatísticas

```
Stack Overflow 2023:
- #1 linguagem mais amada
- #3 mais usada (depois JavaScript/HTML)

GitHub:
- #2 linguagem mais usada
- Crescimento constante

PyPI (Package Index):
- 4.5 milhões de packages
- Crescimento de 15% ao ano
```

---

## 🔗 Referências Cruzadas

- [[Python|Nota principal — Python]]
- [[lang-js-overview|JavaScript]] - Web alternativa
- [[lang-go-overview|Go]] - Backend performático
- [[Conceitos|Ciência da Computação]] - Fundamentos

---

**Próximo:** [[lang-py-syntax|Python Syntax & Types]]

**Status:** ✅ Completo  
**Dificuldade:** ⭐ Iniciante  
**Tempo de Leitura:** 15-20 minutos
