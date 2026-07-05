---
type: technology
id: fw-express-routing
created: 2026-07-05
category: Express
---

# 🚂 Express - Routing
```javascript
app.get('/', (req, res) => res.send('GET'));
app.post('/', (req, res) => res.json(req.body));
app.put('/:id', (req, res) => res.send(`Update ${req.params.id}`));
app.delete('/:id', (req, res) => res.send(`Delete ${req.params.id}`));
```
**Status:** ✅ Completo
