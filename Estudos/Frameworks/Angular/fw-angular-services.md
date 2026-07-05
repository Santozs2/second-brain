---
type: tech
area: Estudos
id: fw-angular-services
category: Angular
created: 2026-07-05
updated: 2026-07-05
---
# 🅰️ Angular - Services & Dependency Injection

---

## 📋 Service

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class DataService {
  constructor(private http: HttpClient) {}
  
  getData(): Observable<any> {
    return this.http.get('/api/data');
  }
}
```

---

## 🔌 Injection

```typescript
@Component({...})
export class MyComponent {
  constructor(private dataService: DataService) {}
  
  ngOnInit() {
    this.dataService.getData().subscribe(data => {
      console.log(data);
    });
  }
}
```

---

## 🎯 Providers

Singleton services available app-wide.

---

**Status:** ✅ Completo
