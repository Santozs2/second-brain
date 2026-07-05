---
type: technology
id: fw-angular-routing
created: 2026-07-05
category: Angular
---

# 🅰️ Angular - Routing

---

## 🛣️ Routes

```typescript
const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: 'user/:id', component: UserComponent },
  { path: '**', component: NotFoundComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)]
})
export class AppModule {}
```

---

## 🔗 Navigation

```html
<a routerLink="/about">About</a>
<router-outlet></router-outlet>
```

```typescript
constructor(private router: Router) {}
navigate() { this.router.navigate(['/about']); }
```

---

## 🛡️ Guards

Protect routes with auth guards.

---

**Status:** ✅ Completo
