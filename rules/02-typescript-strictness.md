# Rule 02: TypeScript Strictness & Typing Standards

> **Category:** Type Safety  
> **Applies To:** All React + TypeScript components, hooks, and services  
> **Enforcement Level:** Zero Tolerance (`noImplicitAny`, `strictNullChecks`, `noUncheckedIndexedAccess`)  

---

## 1. `type` vs `interface`

### Principle
- **Default Rule:** Use `type` by default for all Component Props, Unions, Intersections, Function signatures, and Tuples.
- **Exception:** Use `interface` **only** when defining extensible contracts for object shapes that require declaration merging (e.g., third-party library types or public plugin APIs).

#### ❌ Bad: Inconsistent usage and accidental declaration merging
```typescript
// Accidental declaration merging can inject unexpected fields
interface UserCardProps {
  id: string;
  name: string;
}

// In another file or lower in the codebase...
interface UserCardProps {
  role?: string; // Silently mutates the original interface
}
```

#### ✅ Good: Immutable, predictable type aliases
```typescript
export type UserRole = 'admin' | 'editor' | 'viewer';

export type UserCardProps = {
  readonly id: string;
  readonly name: string;
  readonly role: UserRole;
  readonly onSelect: (id: string) => void;
};
```

---

## 2. Typing Component Props & Children

### Principles
- **No `React.FC`:** Standard functions with explicit typed props provide better type inference, eliminate unwanted default generics, and avoid implicit runtime baggage.
- **No loose `PropsWithChildren`:** `PropsWithChildren` marks `children` as optional (`children?: ReactNode`), which causes silent bugs when children are required.
- **`React.ReactNode`:** Standard renderable slot (primitives, JSX, fragments, null).
- **`React.ReactElement`:** Strict requirement for exactly one JSX element (e.g. icon slot, render delegation).

#### ❌ Bad: Legacy `React.FC` or loose `PropsWithChildren`
```typescript
// Bad: children is silently optional; permits <Modal></Modal> with empty content
const Modal: React.FC<ModalProps> = ({ title, children }) => { ... };

// Bad: children allows undefined even when the component requires content
type CardProps = React.PropsWithChildren<{
  title: string;
}>;
```

#### ✅ Good: Standard function declaration with explicit typing
```typescript
export type ModalProps = {
  readonly title: string;
  readonly children: React.ReactNode; // Required slot
  readonly icon?: React.ReactElement; // Exactly one valid element
};

export function Modal({ title, children, icon }: ModalProps) {
  return (
    <div className="modal">
      <header>
        {icon}
        <h3>{title}</h3>
      </header>
      <main>{children}</main>
    </div>
  );
}
```

---

## 3. Strict Event Handler Typing

### Principle
Never type event parameters as `any`, `unknown`, or generic `Function`. Use React's synthetic event wrappers or pass tailored callback signatures.

#### ❌ Bad: Unsafe and untyped events
```typescript
// Runtime crash risk and loss of IDE autocompletion
const handleChange = (e: any) => {
  console.log(e.target.value);
};

type SearchBarProps = {
  onSubmit: Function;
};
```

#### ✅ Good: Precise SyntheticEvent types and narrowed callback signatures
```typescript
type SearchBarProps = {
  readonly onSearchSubmit: (query: string) => void;
};

export function SearchBar({ onSearchSubmit }: SearchBarProps) {
  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    // e.target is strictly typed as HTMLInputElement
    console.log(e.target.value);
  };

  const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter') {
      onSearchSubmit(e.currentTarget.value);
    }
  };

  return <input type="text" onChange={handleInputChange} onKeyDown={handleKeyDown} />;
}
```

---

## 4. Type Inference vs Explicit Typing

### Principle
- **Infer:** Variable declarations with immediate primitives, simple `useState` defaults, and array transformations.
- **Force Explicit Typing:** API contracts, complex/discriminated union states, custom hook return tuples/objects, and generic helper signatures.

#### ❌ Bad: Over-annotating obvious code while neglecting complex boundaries
```typescript
// Redundant and noisy
const [isOpen, setIsOpen] = useState<boolean>(false);
const count: number = 0;

// Dangerous: unannotated complex state defaults to null (cannot call setUser(actualUser))
const [user, setUser] = useState(null);
```

#### ✅ Good: Pragmatic balance of inference and explicit contracts
```typescript
// Inferred cleanly
const [isOpen, setIsOpen] = useState(false);
const count = 0;

// Explicitly typed complex union / nullable state
type User = {
  readonly id: string;
  readonly email: string;
};

const [user, setUser] = useState<User | null>(null);
```

---

## 5. Strict Typing for Hooks

### `useState` with Discriminated Unions
```typescript
// ✅ Good: Guarantees impossible states cannot be represented
type AsyncState<T> =
  | { readonly status: 'idle'; readonly data: null; readonly error: null }
  | { readonly status: 'loading'; readonly data: null; readonly error: null }
  | { readonly status: 'success'; readonly data: T; readonly error: null }
  | { readonly status: 'error'; readonly data: null; readonly error: Error };

const [state, setState] = useState<AsyncState<User>>({
  status: 'idle',
  data: null,
  error: null,
});
```

### `useReducer` with Exhaustive Type Checking
```typescript
type State = { readonly count: number };

type Action =
  | { readonly type: 'INCREMENT'; readonly step: number }
  | { readonly type: 'DECREMENT'; readonly step: number }
  | { readonly type: 'RESET' };

function counterReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + action.step };
    case 'DECREMENT':
      return { count: state.count - action.step };
    case 'RESET':
      return { count: 0 };
    default: {
      // Compile-time check: triggers build error if an action variant is unhandled
      const _exhaustiveCheck: never = action;
      return state;
    }
  }
}
```

### `useRef`: DOM Reference vs Mutable Instance Value
- **DOM References:** Initialize with `null`, type the element. The `.current` is read-only to external writes.
- **Mutable Values (Timer IDs, previous values):** Pass the initial value directly. `.current` is mutable.

```typescript
// ✅ DOM Reference (read-only current)
const inputRef = useRef<HTMLInputElement>(null);

// ✅ Mutable Value Reference (read-write current)
const timerRef = useRef<number | null>(null);

function handleStart() {
  timerRef.current = window.setInterval(() => {}, 1000);
}
```
