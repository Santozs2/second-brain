---
type: technology
id: lang-js-oop
created: 2026-07-05
updated: 2026-07-05
category: JavaScript
tags:
  - type/technology
  - language/javascript
---

# 🟨 JavaScript - OOP & Prototypes

---

## 🏗️ Prototypal Inheritance

```javascript
// Prototype chain
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  console.log(this.name + " makes sound");
};

function Dog(name) {
  Animal.call(this, name);
}

Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

const dog = new Dog("Rex");
dog.speak(); // Rex makes sound
```

---

## 🎯 ES6 Classes

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    console.log(this.name);
  }
}

class Dog extends Animal {
  speak() {
    console.log(this.name + " barks");
  }
}

const dog = new Dog("Rex");
dog.speak(); // Rex barks
```

---

## 💡 Key Concepts

- Prototype chain
- `this` binding
- Constructors
- Inheritance
- Composition over inheritance

---

**Status:** ✅ Completo
