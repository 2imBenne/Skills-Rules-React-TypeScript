---
name: react-feature-scaffold
description: >-
  Use this skill when scaffolding a new business feature module in a React + TypeScript project.
  Enforces feature colocation, modular boundary encapsulation, and strict public API contracts.
---

# Skill: React Feature Scaffolding Workflow

This workflow standardizes the creation of new domain features, ensuring that all code adheres to colocation and public boundary encapsulation.

---

## 1. Directory Structure Blueprint

When creating a new feature (e.g. `billing`), generate the following structure in `src/features/<feature_name>/`:

```
src/features/<feature_name>/
├── api/
│   ├── use<Feature>Query.ts       # TanStack Query hook
│   └── use<Feature>Mutation.ts    # TanStack Mutation hook
├── components/
│   ├── <Feature>Dashboard.tsx     # Feature orchestration
│   └── <Feature>List.tsx          # Pure presentational component
├── hooks/
│   └── use<Feature>Form.ts        # Headless container hook
├── types/
│   └── index.ts                   # Domain types & DTO schemas
├── utils/
│   └── format<Feature>.ts         # Domain transformations
└── index.ts                       # STRICT PUBLIC API BARREL
```

---

## 2. Step-by-Step Scaffolding Procedure

### Step 1: Define Domain Types (`types/index.ts`)
```typescript
export type InvoiceStatus = 'paid' | 'pending' | 'overdue';

export type InvoiceDTO = {
  readonly id: string;
  readonly amount: number;
  readonly currency: string;
  readonly status: InvoiceStatus;
  readonly dueDate: string;
};
```

### Step 2: Create API Query Hook (`api/use<Feature>Query.ts`)
```typescript
import { useQuery } from '@tanstack/react-query';
import type { InvoiceDTO } from '../types';

async function fetchInvoices(): Promise<readonly InvoiceDTO[]> {
  const res = await fetch('/api/invoices');
  if (!res.ok) throw new Error('Failed to fetch invoices');
  return res.json();
}

export function useInvoicesQuery() {
  return useQuery({
    queryKey: ['invoices'],
    queryFn: fetchInvoices,
    staleTime: 1000 * 60 * 5, // 5 minutes cache
  });
}
```

### Step 3: Create Feature Components (`components/`)
Colocate presentational components and container components.

### Step 4: Export Public API (`index.ts`)
Only export items that external modules are permitted to consume.
```typescript
export { InvoiceDashboard } from './components/InvoiceDashboard';
export { useInvoicesQuery } from './api/useInvoicesQuery';
export type { InvoiceDTO, InvoiceStatus } from './types';
```

---

## 3. Validation Checklist
- [ ] No internal subfolders are directly imported from outside `src/features/<feature_name>/`.
- [ ] TanStack Query manages remote cache; no duplicate `useState` mirrors the query data.
- [ ] All types use `type` instead of `interface`.
