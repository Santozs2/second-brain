---
type: tech
area: Estudos
id: fw-angular-http
category: Angular
created: 2026-07-05
updated: 2026-07-05
---
# 🅰️ Angular - HTTP Client

---

## 📡 GET Request

```typescript
constructor(private http: HttpClient) {}

getUsers() {
  return this.http.get<User[]>('/api/users');
}
```

---

## 📝 POST Request

```typescript
createUser(user: User) {
  return this.http.post<User>('/api/users', user);
}
```

---

## 🔄 Observables

```typescript
component() {
  this.dataService.getUsers().pipe(
    map(users => users.filter(u => u.active)),
    catchError(err => of([]))
  ).subscribe(users => console.log(users));
}
```

---

## ⚠️ Error Handling

Use `catchError` operator.

---

**Status:** ✅ Completo
