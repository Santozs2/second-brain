---
type: technology
id: fw-nodejs-streams
created: 2026-07-05
category: Node.js
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
