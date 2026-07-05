---
type: technology
id: fw-express-testing
created: 2026-07-05
category: Express
---

# 🚂 Express - Testing
```javascript
const request = require('supertest');
test('GET /', async () => {
  const res = await request(app).get('/');
  expect(res.status).toBe(200);
});
```
**Status:** ✅ Completo
