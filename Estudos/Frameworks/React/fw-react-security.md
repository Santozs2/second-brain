---
type: technology
id: fw-react-security
created: 2026-07-05
category: React
---

# ⚛️ React - Security Best Practices

---

## 🔒 XSS Prevention

```javascript
// ❌ Dangerous
<div dangerHTML={userInput} />

// ✅ Safe - React escapes by default
<div>{userInput}</div>

// ✅ Safe for attributes
<img src={imageUrl} alt={userText} />
```

---

## 🛡️ CSRF Protection

```javascript
// Include CSRF token in requests
axios.post('/api/data', data, {
  headers: { 'X-CSRF-Token': csrfToken }
});
```

---

## 📝 Input Validation

```javascript
function Form() {
  const [email, setEmail] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (!email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) {
      alert('Invalid email');
      return;
    }
    // Submit
  };
}
```

---

## 🔑 Authentication

- Use httpOnly cookies
- JWT in secure storage
- Implement logout
- Validate tokens server-side
- Use HTTPS only

---

**Status:** ✅ Completo
