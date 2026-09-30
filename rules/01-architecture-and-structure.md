# Rule 01: Project Architecture & File Structure

> **Category:** Architecture & Structure  
> **Applies To:** All React + TypeScript projects  
> **Enforcement Level:** Mandatory  

---

## 1. Feature-Based Modular Architecture (Colocation Principle)

### Principle
Organize code by business domain/feature rather than technical layer (such as dumping all components in one folder and all hooks in another). Everything related to a single business feature—components, hooks, state, types, queries, utilities—must be colocated within that feature directory.

Shared and truly generic UI elements reside in designated top-level folders.

### Canonical Directory Tree
```
src/
├── app/                  # Application routing, providers, and global layout
├── assets/               # Static assets (images, fonts, global SVGs)
├── components/           # Domain-agnostic UI primitives (Button, Modal, Input)
├── hooks/                # Domain-agnostic utility hooks (useDebounce, useMediaQuery)
├── lib/                  # Third-party client setups (axios, tanstack-query, dayjs)
├── types/                # Global ambient and cross-domain types
├── utils/                # Global pure helper functions
└── features/             # Business modules (Domain-driven)
    ├── auth/
    └── billing/
        ├── api/          # Feature API queries, mutations, DTOs
        ├── components/   # Feature-specific components
        ├── hooks/        # Feature-specific hooks
        ├── types/        # Feature-specific types and schemas
        ├── utils/        # Feature-specific transformations
        └── index.ts      # Public API boundary for the feature
```

---

## 2. Module Boundaries & Strict Public API

### Principle
Internal implementation details of a feature must **never** be imported directly by other features or root components. Every feature exposes a single public contract via an `index.ts` file. Cross-boundary imports must go exclusively through this entrypoint.

#### ❌ Bad: Leaking internal feature structure across boundaries
```typescript
// src/features/dashboard/components/DashboardView.tsx
import { InvoiceSummary } from '@/features/billing/components/InvoiceSummary';
import { useInvoiceQuery } from '@/features/billing/api/useInvoiceQuery';
```

#### ✅ Good: Consuming explicit public boundaries
```typescript
// src/features/billing/index.ts
export { InvoiceSummary } from './components/InvoiceSummary';
export { useBillingStatus } from './hooks/useBillingStatus';
export type { InvoiceSummaryDTO } from './types';

// src/features/dashboard/components/DashboardView.tsx
import { InvoiceSummary, useBillingStatus } from '@/features/billing';
```

---

## 3. Barrel Files Restriction (Bundle & Build Performance)

### Principle
Do **not** create catch-all global barrel files (`src/components/index.ts` exporting 100 components). They degrade bundler compilation speed, compromise tree-shaking, and frequently cause circular dependency graphs.

#### ❌ Bad: Mega barrel file aggregating hundreds of unrelated modules
```typescript
// src/components/index.ts
export * from './Button';
export * from './DataGrid';
export * from './RichTextEditor'; // Forces bundler to parse all imports even if only Button is used
```

#### ✅ Good: Direct import for shared UI or strictly isolated feature barrels
```typescript
// Import direct atomic module
import { Button } from '@/components/ui/Button';
import { DataGrid } from '@/components/ui/DataGrid';
```
