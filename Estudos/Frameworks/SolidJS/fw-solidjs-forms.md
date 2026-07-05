---
type: technology
id: fw-solidjs-forms
created: 2026-07-05
category: SolidJS
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
