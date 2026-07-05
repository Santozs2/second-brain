---
type: tech
area: Estudos
id: fw-svelte-testing
category: Svelte
created: 2026-07-05
updated: 2026-07-05
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
