---
type: technology
id: fw-react-testing
created: 2026-07-05
category: React
---

# ⚛️ React - Testing

---

## 🧪 Unit Testing

```javascript
import { render, screen } from '@testing-library/react';
import Button from './Button';

test('renders button', () => {
  render(<Button label="Click" />);
  expect(screen.getByText('Click')).toBeInTheDocument();
});
```

---

## 🎯 Component Testing

```javascript
test('updates on click', () => {
  const { rerender } = render(<Counter />);
  fireEvent.click(screen.getByRole('button'));
  expect(screen.getByText('1')).toBeInTheDocument();
});
```

---

## 🔗 Integration Testing

```javascript
test('user flow', async () => {
  render(<App />);
  fireEvent.click(screen.getByText('Login'));
  await waitFor(() => {
    expect(screen.getByText('Dashboard')).toBeInTheDocument();
  });
});
```

---

## 🛠️ Tools

- **Jest** - Test runner
- **React Testing Library** - Component testing
- **Vitest** - Fast test runner
- **Cypress** - E2E testing
- **Playwright** - Browser automation

---

**Status:** ✅ Completo
