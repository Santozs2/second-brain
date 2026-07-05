---
type: technology
id: fw-svelte-testing
created: 2026-07-05
category: Svelte
---

# 🔥 Svelte - Testing

---

## 🧪 Unit Testing

```javascript
import { render, screen } from '@testing-library/svelte';
import Component from './Component.svelte';

test('renders', () => {
  render(Component);
  expect(screen.getByText('Hello')).toBeInTheDocument();
});
```

---

## 🎯 Tools

- Vitest
- Testing Library
- Playwright

---

**Status:** ✅ Completo
