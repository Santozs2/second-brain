---
type: technology
id: fw-angular-testing
created: 2026-07-05
category: Angular
---

# 🅰️ Angular - Testing

---

## 🧪 Unit Testing

```typescript
import { TestBed } from '@angular/core/testing';
import { MyComponent } from './my.component';

describe('MyComponent', () => {
  let component: MyComponent;
  let fixture: ComponentFixture<MyComponent>;
  
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [MyComponent]
    }).compileComponents();
    
    fixture = TestBed.createComponent(MyComponent);
    component = fixture.componentInstance;
  });
  
  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

---

## 🔍 Testing Services

Mock dependencies with Jasmine spies.

---

## E2E Testing

Use Cypress or Protractor.

---

**Status:** ✅ Completo
