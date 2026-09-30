# Rule 03: Component Design Patterns

> **Category:** Component Architecture & SOLID Principles  
> **Applies To:** Functional Components and Custom Hooks  
> **Enforcement Level:** Mandatory  

---

## 1. Single Responsibility Principle (SRP)

### Principle
A component must do one thing: render a piece of UI, compose a set of widgets, or orchestrate an interaction. Never mix data fetching, API transformations, complex validation, and rendering in a single monolithic component.

#### ❌ Bad: Monolith component doing network, mapping, state, and presentation
```typescript
export function OrderDashboard() {
  const [data, setData] = useState<any[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/orders')
      .then((res) => res.json())
      .then((items) => {
        setData(items.map((i: any) => ({ ...i, formattedTotal: `$${i.total.toFixed(2)}` })));
        setLoading(false);
      });
  }, []);

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      {data.map((order) => (
        <div key={order.id}>{order.formattedTotal}</div>
      ))}
    </div>
  );
}
```

#### ✅ Good: Orchestration through separation of concerns
```typescript
// 1. Hook encapsulates fetch & domain transformation
export function useOrders() {
  return useQuery({
    queryKey: ['orders'],
    queryFn: fetchOrders,
    select: (orders) => orders.map(formatOrderSummary),
  });
}

// 2. Pure UI Presentational component
export function OrderList({ orders }: { readonly orders: readonly FormattedOrder[] }) {
  return (
    <ul>
      {orders.map((order) => (
        <li key={order.id}>{order.formattedTotal}</li>
      ))}
    </ul>
  );
}

// 3. Orchestration component
export function OrderDashboard() {
  const { data: orders, isLoading, isError } = useOrders();

  if (isLoading) return <OrderSkeleton />;
  if (isError || !orders) return <ErrorMessage message="Failed to load orders." />;

  return <OrderList orders={orders} />;
}
```

---

## 2. Container / Presentational Pattern (Modern Headless Hooks)

### Principle
Decouple state logic and side-effects from pure rendering. Use Custom Hooks as "Headless Containers" and pure functional components as "Presentational Views".

#### ✅ Good: Headless Hook + Presentational View
```typescript
// useUserProfileForm.ts (Headless Container)
export function useUserProfileForm(userId: string) {
  const { data: user } = useUserQuery(userId);
  const mutation = useUpdateUserMutation();
  const [name, setName] = useState(user?.name ?? '');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    await mutation.mutateAsync({ id: userId, name });
  };

  return {
    name,
    setName,
    handleSubmit,
    isSubmitting: mutation.isPending,
  };
}

// UserProfileView.tsx (Presentational View)
type UserProfileViewProps = ReturnType<typeof useUserProfileForm>;

export function UserProfileView({ name, setName, handleSubmit, isSubmitting }: UserProfileViewProps) {
  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Saving...' : 'Save'}
      </button>
    </form>
  );
}
```

---

## 3. Composition Over Configuration

### Principle
Avoid creating monolithic components controlled by a maze of conditional boolean flags (`hasIcon`, `showFooter`, `isCompact`, `withBadge`). Use Compound Components or Slot Props.

#### ❌ Bad: Boolean prop explosion (Configuration Hell)
```typescript
<Card
  title="Profile"
  showAvatar
  avatarUrl="/avatar.png"
  hasActions
  actionButtonText="Edit"
  onActionClick={handleEdit}
  footerText="Last active 2m ago"
/>
```

#### ✅ Good: Composable compound structure
```typescript
export function Card({ children }: { readonly children: React.ReactNode }) {
  return <div className="card-container">{children}</div>;
}

Card.Header = function CardHeader({ children }: { readonly children: React.ReactNode }) {
  return <div className="card-header">{children}</div>;
};

Card.Body = function CardBody({ children }: { readonly children: React.ReactNode }) {
  return <div className="card-body">{children}</div>;
};

Card.Footer = function CardFooter({ children }: { readonly children: React.ReactNode }) {
  return <div className="card-footer">{children}</div>;
};

// Usage: Infinite flexibility with zero prop proliferation
<Card>
  <Card.Header>
    <Avatar src="/avatar.png" />
    <h3>Profile</h3>
  </Card.Header>
  <Card.Body>User detailed information goes here.</Card.Body>
  <Card.Footer>
    <Button onClick={handleEdit}>Edit</Button>
  </Card.Footer>
</Card>
```

---

## 4. Safe Context API Usage

### Principles
- Use React Context exclusively for low-frequency global values: Themes, Auth sessions, Localization, or Compound Component coordination.
- **Never** use Context as a high-frequency global event bus (triggers entire subtree re-renders).
- Split State and Dispatch into separate contexts to insulate consumers from updates.
- Always provide a custom consumer hook with an invariant check.

#### ✅ Good: Segregated context with fail-fast consumer hook
```typescript
type Theme = 'light' | 'dark';
type ThemeDispatch = (theme: Theme) => void;

const ThemeStateContext = createContext<Theme | undefined>(undefined);
const ThemeDispatchContext = createContext<ThemeDispatch | undefined>(undefined);

export function ThemeProvider({ children }: { readonly children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');

  return (
    <ThemeStateContext.Provider value={theme}>
      <ThemeDispatchContext.Provider value={setTheme}>
        {children}
      </ThemeDispatchContext.Provider>
    </ThemeStateContext.Provider>
  );
}

// Invariant-guarded custom hooks
export function useTheme(): Theme {
  const context = useContext(ThemeStateContext);
  if (context === undefined) {
    throw new Error('useTheme must be consumed within a ThemeProvider');
  }
  return context;
}

export function useSetTheme(): ThemeDispatch {
  const context = useContext(ThemeDispatchContext);
  if (context === undefined) {
    throw new Error('useSetTheme must be consumed within a ThemeProvider');
  }
  return context;
}
```
