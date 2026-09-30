# Rule 05: Performance & Memoization Standards

> **Category:** Runtime Optimization & Render Cycles  
> **Applies To:** `useMemo`, `useCallback`, `React.memo`  
> **Enforcement Level:** Mandatory  

---

## 1. The Cost of Memoization

Memoization is not free: each call to `useMemo`, `useCallback`, and `React.memo` introduces memory allocations for dependency arrays and shallow equality checking on every render. Blindly memoizing everything causes a net loss in performance.

---

## 2. Strict Memoization Decision Matrix

| Mechanism | Use When | DO NOT Use When |
| :--- | :--- | :--- |
| **`React.memo`** | Heavy UI subtrees that re-render frequently with identical props while parent re-renders. | Component is cheap to render, or props change on almost every render. |
| **`useCallback`** | 1. Passing callback to a child wrapped in `React.memo`<br>2. Function is a dependency in `useEffect`. | Passing callback to native DOM elements (`<button onClick={...}>`). |
| **`useMemo`** | 1. Computationally expensive loops (e.g. 10k items sort/filter)<br>2. Stabilizing referential object passed to `React.memo` or hook dependency. | Primitive transforms, string concatenation, or lightweight filtering (< 100 items). |

---

## 3. Code Patterns

#### ❌ Bad: Senseless micro-optimizations that degrade performance
```typescript
// 1. Primitive calculation wrapped in useMemo (Overhead > Computation)
const fullName = useMemo(() => `${firstName} ${lastName}`, [firstName, lastName]);

// 2. useCallback passed directly to native HTML tag (Useless)
const handleClick = useCallback(() => {
  console.log('Clicked');
}, []);

return <button onClick={handleClick}>{fullName}</button>;
```

#### ✅ Good: Justified memoization targeting referential stability and heavy trees
```typescript
// Expensive component wrapped in memo
const DataGrid = React.memo(function DataGrid({
  rows,
  onRowClick,
}: {
  readonly rows: readonly Row[];
  readonly onRowClick: (id: string) => void;
}) {
  return <div>{/* 5,000 DOM nodes rendered here */}</div>;
});

export function ParentDashboard({ data }: { readonly data: readonly RawItem[] }) {
  // Heavy computation stabilized with useMemo
  const processedRows = useMemo(() => {
    return transformAndSortThousandsOfRows(data);
  }, [data]);

  // Referential identity preserved for React.memo child
  const handleRowClick = useCallback((id: string) => {
    trackRowAnalytics(id);
  }, []);

  return <DataGrid rows={processedRows} onRowClick={handleRowClick} />;
}
```
