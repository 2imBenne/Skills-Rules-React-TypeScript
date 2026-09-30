# Rule 06: Testing Standards (Behavior-Driven)

> **Category:** Quality Assurance & Testing  
> **Applies To:** Component Tests, Integration Tests  
> **Frameworks:** React Testing Library + `@testing-library/user-event`  
> **Enforcement Level:** Mandatory  

---

## 1. Guiding Principle: Test Behavior, Not Implementation Details

- Test what the user sees, hears (assistive tech), and interacts with.
- Do **not** test component internal state, hook variable values, or internal function calls.
- Avoid querying by CSS class name or arbitrary HTML tags.
- Follow the accessibility tree query hierarchy.

---

## 2. Query Priority Hierarchy

1. **`getByRole`** (Simulates how assistive technology and real users perceive the page)
2. **`getByLabelText`** (For form inputs and controls)
3. **`getByPlaceholderText`**
4. **`getByText`**
5. **`getByTestId`** (**Strict last resort** only when no accessible semantic selector exists)

---

## 3. Test Implementation Example

#### ❌ Bad: Brittle, implementation-focused test
```typescript
test('increments counter on button click', () => {
  const wrapper = render(<Counter />);
  
  // Implementation details: relies on CSS classes and internal structure
  const button = wrapper.container.querySelector('.btn-primary');
  fireEvent.click(button!);
  
  // Inspecting raw HTML instead of accessible semantic output
  const span = wrapper.container.querySelector('span');
  expect(span?.textContent).toBe('Count: 1');
});
```

#### ✅ Good: Resilient, accessible, behavior-driven test
```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Counter } from './Counter';

describe('Counter Component', () => {
  it('increments the count and updates aria-live status when user activates button', async () => {
    const user = userEvent.setup();
    render(<Counter initialCount={0} />);

    // Query by explicit accessible role and name
    const incrementButton = screen.getByRole('button', { name: /increment count/i });
    const countDisplay = screen.getByRole('status');

    expect(countDisplay).toHaveTextContent(/current count: 0/i);

    // Realistic user interaction with userEvent
    await user.click(incrementButton);

    expect(countDisplay).toHaveTextContent(/current count: 1/i);
  });
});
```
