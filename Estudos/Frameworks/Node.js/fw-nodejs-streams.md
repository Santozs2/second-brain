---
type: tech
area: Estudos
id: fw-nodejs-streams
category: Node.js
created: 2026-07-05
updated: 2026-07-05
---
# 🟢 Node.js - Streams

---

## 🌊 Readable Stream

```javascript
fs.createReadStream('file.txt')
  .on('data', (chunk) => console.log(chunk))
  .on('end', () => console.log('done'));
```

---

## 📝 Piping

```javascript
fs.createReadStream('in.txt')
  .pipe(fs.createWriteStream('out.txt'));
```

---

**Status:** ✅ Completo
