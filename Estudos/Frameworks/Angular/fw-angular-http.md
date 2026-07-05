---
type: technology
id: fw-angular-http
created: 2026-07-05
category: Angular
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
