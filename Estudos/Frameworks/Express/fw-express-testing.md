---
type: tech
area: Estudos
id: fw-express-testing
category: Express
created: 2026-07-05
updated: 2026-07-05
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
