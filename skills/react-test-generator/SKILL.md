---
name: react-test-generator
description: >-
  Use this skill when generating behavior-driven unit and integration tests for React components.
  Enforces React Testing Library best practices, accessible query hierarchies, and user-event simulation.
---

# Skill: React Behavior-Driven Test Generator Workflow

This workflow enforces the creation of resilient, accessible, behavior-focused tests using React Testing Library and `@testing-library/user-event`.

---

## 1. Test Architecture: Arrange-Act-Assert (AAA)

All tests must be strictly partitioned into three readable phases:

```typescript
it('submits form with valid user credentials', async () => {
  // 1. Arrange
  const user = userEvent.setup();
  const handleSubmit = vi.fn();
  render(<LoginForm onSubmit={handleSubmit} />);

  const emailInput = screen.getByRole('textbox', { name: /email address/i });
  const submitButton = screen.getByRole('button', { name: /log in/i });

  // 2. Act
  await user.type(emailInput, 'developer@enterprise.com');
  await user.click(submitButton);

  // 3. Assert
  expect(handleSubmit).toHaveBeenCalledTimes(1);
  expect(handleSubmit).toHaveBeenCalledWith({ email: 'developer@enterprise.com' });
});
```

---

## 2. Accessible Query Decision Tree

Choose your selector using this strict order of precedence:

1. Can the element be reached via accessibility role (`button`, `dialog`, `textbox`, `heading`, `status`)?
   👉 Use `screen.getByRole(role, { name: /pattern/i })`.
2. Is it an input control with an associated label?
   👉 Use `screen.getByLabelText(/label text/i)`.
3. Is it non-semantic body text or notification toast?
   👉 Use `screen.getByText(/message text/i)`.
4. Is it a dynamic element that will appear asynchronously?
   👉 Use `await screen.findByRole(...)` (never `waitFor` with `getBy*`).
5. Is it completely non-semantic and unreachable via standard ARIA attributes?
   👉 Only then use `screen.getByTestId('explicit-id')`.

---

## 3. Strict Prohibitions
- ❌ **No `fireEvent`:** Always use `userEvent.setup()` and `await user.<action>()`.
- ❌ **No querying internal state or props.**
- ❌ **No querying CSS selectors (`container.querySelector`)**.
