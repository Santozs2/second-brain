---
type: tech
area: Estudos
id: fw-angular-forms
category: Angular
created: 2026-07-05
updated: 2026-07-05
---
# 🅰️ Angular - Forms

---

## 📝 Reactive Forms

```typescript
import { FormBuilder, FormGroup } from '@angular/forms';

export class MyComponent {
  form: FormGroup;
  
  constructor(fb: FormBuilder) {
    this.form = fb.group({
      name: ['', Validators.required],
      email: ['', Validators.email]
    });
  }
  
  submit() {
    if (this.form.valid) {
      console.log(this.form.value);
    }
  }
}
```

---

## 🎯 Template Forms

```html
<form (ngSubmit)="submit()">
  <input [(ngModel)]="name" name="name">
  <button type="submit">Submit</button>
</form>
```

---

## ✅ Validation

Built-in: required, email, pattern, minlength, maxlength.

---

**Status:** ✅ Completo
