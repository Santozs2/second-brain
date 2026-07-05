---
type: tech
area: Estudos
id: fw-angular-directives
category: Angular
created: 2026-07-05
updated: 2026-07-05
---
# 🅰️ Angular - Directives

---

## 🎯 Structural Directives

```html
<div *ngIf="isVisible">Visible</div>
<div *ngFor="let item of items">{{ item }}</div>
<div [ngSwitch]="status">
  <div *ngSwitchCase="'active'">Active</div>
  <div *ngSwitchDefault>Inactive</div>
</div>
```

---

## 📝 Attribute Directives

```html
<div [ngClass]="{ active: isActive }">Classes</div>
<div [ngStyle]="{ color: 'red' }">Style</div>
<input [(ngModel)]="name">
```

---

## 🔧 Custom Directives

```typescript
@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  constructor(el: ElementRef) {
    el.nativeElement.style.backgroundColor = 'yellow';
  }
}
```

---

**Status:** ✅ Completo
