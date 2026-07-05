---
type: tech
area: Estudos
id: fw-angular-components
category: Angular
created: 2026-07-05
updated: 2026-07-05
---
# 🅰️ Angular - Components

---

## 📝 Basic Component

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-hello',
  template: '<h1>{{ title }}</h1>',
  styles: ['h1 { color: blue; }']
})
export class HelloComponent {
  title = 'Hello Angular';
}
```

---

## 🔗 Input & Output

```typescript
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-button',
  template: '<button (click)="onClick()">{{ label }}</button>'
})
export class ButtonComponent {
  @Input() label = 'Click';
  @Output() clicked = new EventEmitter<void>();
  
  onClick() { this.clicked.emit(); }
}
```

---

## 🎯 Lifecycle Hooks

`OnInit`, `OnDestroy`, `OnChanges`, etc.

---

**Status:** ✅ Completo
