---
type: tech
area: Estudos
id: fw-express-routing
category: Express
created: 2026-07-05
updated: 2026-07-05
---
# 🚂 Express - Routing
```javascript
app.get('/', (req, res) => res.send('GET'));
app.post('/', (req, res) => res.json(req.body));
app.put('/:id', (req, res) => res.send(`Update ${req.params.id}`));
app.delete('/:id', (req, res) => res.send(`Delete ${req.params.id}`));
```
**Status:** ✅ Completo
