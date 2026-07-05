---
type: tech
area: Estudos
status: stable
difficulty: beginner
id: lang-js-overview
category: JavaScript
tags:
  - frontend
  - backend
  - javascript
created: 2026-07-04
updated: 2026-07-04
---
# 🟨 JavaScript - Overview & History

> The Programming Language of the Web - From Browser to Everywhere

---

## 📖 Definição

**JavaScript** é:
- **Interpretada/JIT-compilada** - Executada em runtime
- **Dinâmica** - Tipos determinados em runtime
- **Prototypal** - OOP baseado em protótipos
- **Funcional** - First-class functions
- **Assincronismo** - Callbacks, Promises, Async/Await
- **Event-driven** - Baseada em eventos

```
Runtime Environments:
┌──────────────────────────────────┐
│   Browser (Chrome, Firefox)      │
├──────────────────────────────────┤
│   Node.js (Backend)              │
├──────────────────────────────────┤
│   Deno (Moderno, TypeScript)     │
├──────────────────────────────────┤
│   Bun (Rápido)                   │
├──────────────────────────────────┤
│   Electron (Desktop)             │
├──────────────────────────────────┤
│   React Native (Mobile)          │
└──────────────────────────────────┘
```

---

## 📜 História

### **1995 - Inception (Brendan Eich)**
```
May 1995: Criado em 10 dias por Brendan Eich
Nome original: "Mocha" → "LiveScript" → "JavaScript"
Razão do nome: Parceria Netscape-Sun (Java popular)
```

### **Versões Principais**

#### ECMAScript 1 (1997)
- Primeira padronização
- Especificação ECMA-262

#### ES3 (1999)
- Try/catch, regular expressions
- Padrão estável por 10 anos

#### ES5 (2009)
```javascript
// Strict mode
'use strict';

// Object methods
Object.defineProperty
Object.keys()

// Array methods
[].forEach, [].map, [].filter
```

#### **ES6/ES2015 - Grande Revolução** 🔥
```javascript
// Arrow functions
const add = (a, b) => a + b;

// Classes
class User {
  constructor(name) {
    this.name = name;
  }
}

// Promises
new Promise((resolve, reject) => {})

// Const/let (block scope)
const x = 1;

// Template literals
const msg = `Hello ${name}`;

// Destructuring
const { x, y } = obj;

// Spread operator
const arr = [...arr1, ...arr2];
```

#### ES2016+ (Anual releases)
```
ES2016 → Array.includes()
ES2017 → Async/Await
ES2018 → Rest parameters, Spread in objects
ES2019 → Optional chaining (?.)
ES2020 → Nullish coalescing (??)
ES2021 → Logical assignment
ES2022 → Top-level await
```

### Timeline Visual
```
1995: Birth (Mocha)
  │
1997: ES1
  │
1999: ES3 (estável 10 anos)
  │
2009: ES5 (strict mode, map/filter)
  │
2015: ES6/ES2015 (classes, promises, arrow functions)
  │
2016+: Anual updates (async/await, optional chaining)
  │
2024: ECMAScript 2024
```

---

## 🌍 Onde JavaScript Roda

### **Browser**
```
┌─────────────────────────┐
│  Chrome V8 Engine       │
│  Firefox SpiderMonkey   │
│  Safari JavaScriptCore  │
│  Edge Chakra            │
└─────────────────────────┘
```

### **Backend (Node.js)**
```javascript
// server.js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(3000);
```

### **Desktop (Electron)**
```javascript
// main.js
const { app, BrowserWindow } = require('electron');

app.on('ready', () => {
  const win = new BrowserWindow();
  win.loadFile('index.html');
});
```

### **Mobile (React Native)**
```javascript
// app.js
import React from 'react';
import { Text, View } from 'react-native';

export default function App() {
  return (
    <View>
      <Text>Hello Mobile!</Text>
    </View>
  );
}
```

---

## 🎯 Paradigmas Suportados

### 1. **Orientado a Objetos (OOP)**
```javascript
// Classes (ES6+)
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    console.log(`${this.name} makes a sound`);
  }
}

const dog = new Animal('Dog');
dog.speak(); // Dog makes a sound
```

### 2. **Funcional**
```javascript
// Pure functions
const add = (a, b) => a + b;

// Array methods
const nums = [1, 2, 3, 4, 5];
const doubled = nums
  .map(x => x * 2)
  .filter(x => x > 4)
  .reduce((sum, x) => sum + x, 0);

// Higher-order functions
const multiplier = (n) => (x) => x * n;
const double = multiplier(2);
double(5); // 10
```

### 3. **Event-Driven**
```javascript
// Button click
const btn = document.getElementById('btn');
btn.addEventListener('click', () => {
  console.log('Clicked!');
});

// DOM events
window.addEventListener('resize', () => {
  console.log('Window resized');
});
```

### 4. **Assincronismo**
```javascript
// Callbacks (antiguo)
setTimeout(() => {
  console.log('After 1 second');
}, 1000);

// Promises
fetch('/api/data')
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err));

// Async/Await (moderno)
async function getData() {
  try {
    const res = await fetch('/api/data');
    const data = await res.json();
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}
```

---

## 💪 Strengths

✅ **Ubíquo** - Roda em qualquer navegador  
✅ **Ecosistema** - npm, 2M+ pacotes  
✅ **Frameworks** - React, Vue, Angular  
✅ **Curva de aprendizado** - Relativamente fácil  
✅ **Comunidade** - Enorme, ativa  
✅ **Full-stack** - Frontend + Backend  
✅ **Event-driven** - Natural para UI  

---

## ⚠️ Weaknesses

❌ **Dinâmica** - Erros em runtime  
❌ **Performance** - Mais lenta que C/Rust/Go  
❌ **Memory leaks** - GC nem sempre perfeito  
❌ **Async complexity** - Callback hell, Promise hell  
❌ **Coerção de tipos** - Comportamentos inesperados  

---

## 🔄 Execution Model

### **Single-threaded Event Loop**
```
┌─────────────────────────────────┐
│     Call Stack                  │
│  ┌─────────────────────────┐   │
│  │ Script executing        │   │
│  └─────────────────────────┘   │
└─────────────────────────────────┘
            ↓
┌─────────────────────────────────┐
│     Event Loop                  │
│  - Microqueue (Promises)        │
│  - Macroqueue (setTimeout)      │
│  - I/O operations               │
└─────────────────────────────────┘
```

### **Exemplo Prático**
```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve()
  .then(() => console.log('3'));

console.log('4');

// Output:
// 1
// 4
// 3 (Microtask - Promise)
// 2 (Macrotask - setTimeout)
```

---

## 📊 Comparação com Linguagens

| Aspecto | JavaScript | Python | Go | Rust |
|---------|-----------|--------|-----|------|
| **Tipo** | Dinâmica | Dinâmica | Estática | Estática |
| **Performance** | Média | Média | Rápida | Muito Rápida |
| **Web** | Nativa | Framework | Backend | WebAssembly |
| **Curva aprendizado** | Fácil | Fácil | Média | Difícil |
| **Comunidade** | Massiva | Científica | Backend | Growing |

---

## 🚀 Use Cases

### ✅ **Perfeito para**
- Web applications (frontend)
- Server-side backend (Node.js)
- Real-time apps (WebSockets)
- Prototipagem rápida
- Full-stack development
- CLI tools

### ❌ **Não recomendado**
- Sistemas operacionais
- Drivers de hardware
- Aplicações críticas de performance
- Processamento científico pesado
- Embarcado (use Lua/C)

---

## 📈 Estatísticas

```
Stack Overflow Survey 2023:
- 62% dos desenvolvedores usam JavaScript
- #1 linguagem mais usada
- #1 em web development

NPM Registry:
- 2.5 milhões de pacotes
- Crescimento de 10% ao ano

Frameworks Populares:
- React: 43% de adoção
- Vue: 25%
- Angular: 20%
```

---

## 🔗 Referências Cruzadas

- [[JavaScript|Nota principal — JavaScript]]

### Linguagens Relacionadas
- [[TypeScript|TypeScript]] - JS com tipos estáticos
- [[lang-py-overview|Python]] - Backend alternativo

### Conceitos
- [[lang-js-async|Asincronismo]]
- [[lang-js-async|Event Loop]]
- [[lang-js-functional|Closures]]
- [[lang-js-oop|Prototypal Inheritance]]

### Frameworks
- [[React|React]] - UI library
- [[Vue|Vue]] - Progressive framework
- [[Express|Express.js]] - Backend framework

### Ferramentas
- [[Node.js|Node.js]] - Runtime
- [[NPM|NPM]] - Package manager
- [[Webpack|Webpack]] - Bundler
- [[Babel|Babel]] - Transpiler

---

## 📚 Recursos Oficiais

- **MDN Web Docs:** https://developer.mozilla.org/
- **ECMAScript Spec:** https://www.ecma-international.org/
- **Node.js Docs:** https://nodejs.org/

---

**Próximo:** [[lang-js-syntax|Sintaxe & Tipos JavaScript]]

**Status:** ✅ Completo  
**Dificuldade:** ⭐ Iniciante  
**Tempo de Leitura:** 15-20 minutos
