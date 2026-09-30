---
name: react-component-designer
description: >-
  Use this skill when designing a robust, accessible, enterprise-grade React component.
  Enforces Single Responsibility Principle (SRP), Headless Container hooks, and Compound Components.
---

# Skill: React Component Designer Workflow

This workflow provides a standardized protocol for designing enterprise React components, preventing prop explosion and spaghetti business logic.

---

## 1. Design Decision Tree

Before writing JSX, evaluate the component against this matrix:

1. **Does the component fetch remote data or manage non-trivial state?**
   - **YES:** Extract logic into a headless Custom Hook (`use<Component>.ts`). Keep JSX purely presentational.
   - **NO:** Keep it as a pure presentational component.
2. **Does the component have more than 3-4 boolean configuration props?**
   - **YES:** Refactor into a **Compound Component** (`<Card.Header>`, `<Card.Body>`) or **Slot Props**.
   - **NO:** Standard props are acceptable.
3. **Are children required?**
   - **YES:** Explicitly type `children: React.ReactNode` (never `PropsWithChildren`).

---

## 2. Standard Implementation Template

### Headless Hook (`useDialog.ts`)
```typescript
import { useState, useCallback } from 'react';

export function useDialog(defaultOpen = false) {
  const [isOpen, setIsOpen] = useState(defaultOpen);

  const open = useCallback(() => setIsOpen(true), []);
  const close = useCallback(() => setIsOpen(false), []);
  const toggle = useCallback(() => setIsOpen((prev) => !prev), []);

  return { isOpen, open, close, toggle };
}
```

### Compound UI Component (`Dialog.tsx`)
```typescript
import React, { createContext, useContext } from 'react';

type DialogContextValue = {
  readonly isOpen: boolean;
  readonly close: () => void;
};

const DialogContext = createContext<DialogContextValue | undefined>(undefined);

function useDialogContext() {
  const ctx = useContext(DialogContext);
  if (!ctx) throw new Error('Dialog compound subcomponents must be used within <Dialog>');
  return ctx;
}

export type DialogProps = {
  readonly isOpen: boolean;
  readonly onClose: () => void;
  readonly children: React.ReactNode;
};

export function Dialog({ isOpen, onClose, children }: DialogProps) {
  if (!isOpen) return null;

  return (
    <DialogContext.Provider value={{ isOpen, close: onClose }}>
      <div className="dialog-backdrop" role="presentation" onClick={onClose}>
        <div
          role="dialog"
          aria-modal="true"
          className="dialog-content"
          onClick={(e) => e.stopPropagation()}
        >
          {children}
        </div>
      </div>
    </DialogContext.Provider>
  );
}

Dialog.Header = function DialogHeader({ children }: { readonly children: React.ReactNode }) {
  return <header className="dialog-header">{children}</header>;
};

Dialog.Body = function DialogBody({ children }: { readonly children: React.ReactNode }) {
  return <div className="dialog-body">{children}</div>;
};

Dialog.CloseButton = function DialogCloseButton() {
  const { close } = useDialogContext();
  return (
    <button type="button" aria-label="Close dialog" onClick={close}>
      ✕
    </button>
  );
};
```
