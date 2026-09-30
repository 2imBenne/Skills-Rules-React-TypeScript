# Rule 04: State Management Rules

> **Category:** State Architecture & Data Flow  
> **Applies To:** Server State, Client Stores, and Local State  
> **Enforcement Level:** Mandatory  

---

## 1. Strict Boundary: Server State vs Client State

### Principles
- **Server State:** Asynchronous, cached, remote-owned, shared across sessions. Use **TanStack Query** (React Query) or RTK Query.
- **Client State:** Synchronous UI state (modal open/close, active wizard step, draft filters). Use `useState`, `useReducer`, or lightweight stores like **Zustand**.
- **Rule:** Never duplicate server state in local `useState`. Treat TanStack Query cache as the single source of truth for remote data.

#### ❌ Bad: Copying server state into local state
```typescript
function UserProfile({ userId }: { userId: string }) {
  const { data: user } = useQuery({ queryKey: ['user', userId], queryFn: () => fetchUser(userId) });
  
  // Anti-pattern: Stale copy of server state! Desyncs on background refetches.
  const [name, setName] = useState('');

  useEffect(() => {
    if (user) setName(user.name);
  }, [user]);

  return <div>{name}</div>;
}
```

#### ✅ Good: Consume server cache directly; isolate local mutations to form drafts
```typescript
function UserProfile({ userId }: { readonly userId: string }) {
  const { data: user } = useUserQuery(userId);

  // Directly consume cache; no duplicated state
  if (!user) return <Skeleton />;
  return <h1>{user.name}</h1>;
}
```

---

## 2. Derived State over Redundant State

### Principle
Never store in state what can be synchronously calculated during render. Mirroring data into state and syncing it with `useEffect` causes dual renders, race conditions, and synchronization bugs.

#### ❌ Bad: Syncing derived state with `useEffect`
```typescript
function ProductList({ products }: { products: Product[] }) {
  const [filteredProducts, setFilteredProducts] = useState<Product[]>([]);
  const [search, setSearch] = useState('');

  // Redundant render pass and state duplication!
  useEffect(() => {
    setFilteredProducts(
      products.filter((p) => p.name.toLowerCase().includes(search.toLowerCase()))
    );
  }, [products, search]);

  return (...);
}
```

#### ✅ Good: Compute on the fly during render
```typescript
function ProductList({ products }: { readonly products: readonly Product[] }) {
  const [search, setSearch] = useState('');

  // Computed synchronously during render. Fast and zero sync overhead.
  const filteredProducts = products.filter((p) =>
    p.name.toLowerCase().includes(search.toLowerCase())
  );

  return (...);
}
```
