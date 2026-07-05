---
type: tech
area: Estudos
id: fw-solidjs-forms
category: SolidJS
created: 2026-07-05
updated: 2026-07-05
---
# 🟦 SolidJS - Forms

---

## 📝 Form Handling

```javascript
function Form() {
  const [formData, setFormData] = createSignal({ name: '', email: '' });
  
  return (
    <form onsubmit={(e) => {
      e.preventDefault();
      console.log(formData());
    }}>
      <input 
        value={formData().name}
        oninput={(e) => setFormData({...formData(), name: e.target.value})}
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## ✅ Validation

Manual validation with signals.

---

**Status:** ✅ Completo
