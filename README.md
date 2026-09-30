# Skills & Rules: React + TypeScript — The Definitive Engineering Standard

> **Standard Version:** 2.0.0  
> **Target Environment:** React 18+ / 19 | TypeScript 5+ | Next.js (App Router) / Vite  
> **Philosophy:** Strict Type Safety, Single Responsibility, Zero Fluff, Relentless Performance.

---

## Overview

This repository standardizes enterprise frontend engineering for React + TypeScript projects. Rather than maintaining a single monolithic document, it is structured into three specialized domains conforming to the modern AI Agent & Engineering Customization architecture:

```
Skills & Rules React + TypeScript/
├── README.md               # Master index & engineering standards hub
├── agents/                 # Specialist AI agent personas & system prompts
│   ├── react-staff-architect.md
│   └── react-code-reviewer.md
├── rules/                  # Discrete, enforceable technical rules & constraints
│   ├── 01-architecture-and-structure.md
│   ├── 02-typescript-strictness.md
│   ├── 03-component-design.md
│   ├── 04-state-management.md
│   ├── 05-performance-memoization.md
│   └── 06-testing-standards.md
└── skills/                 # Procedural, step-by-step operational workflows
    ├── react-feature-scaffold/
    │   └── SKILL.md
    ├── react-component-designer/
    │   └── SKILL.md
    ├── react-performance-audit/
    │   └── SKILL.md
    └── react-test-generator/
        └── SKILL.md
```

---

## 1. Rules (Technical Constraints & Standards)

Each rule file provides strict guidelines and concrete `❌ Bad` vs `✅ Good` code examples:

| Rule | Title | Primary Focus |
| :--- | :--- | :--- |
| [Rule 01](./rules/01-architecture-and-structure.md) | **Architecture & File Structure** | Feature-based colocation, strict public `index.ts` boundaries, anti-barrel policy. |
| [Rule 02](./rules/02-typescript-strictness.md) | **TypeScript Strictness** | `type` vs `interface`, children typing, typed events, discriminated unions for hooks. |
| [Rule 03](./rules/03-component-design.md) | **Component Design Patterns** | SRP, Headless Container hooks, Compound Components, Context API segregation. |
| [Rule 04](./rules/04-state-management.md) | **State Management** | Server state (TanStack Query) vs Client state, derived state, zero `useEffect` sync. |
| [Rule 05](./rules/05-performance-memoization.md) | **Performance & Memoization** | Memoization decision matrix (`useMemo`, `useCallback`, `React.memo`), avoiding micro-optimizations. |
| [Rule 06](./rules/06-testing-standards.md) | **Testing Standards** | Behavior-driven tests, RTL query priority (`getByRole`), `@testing-library/user-event`. |

---

## 2. Skills (Operational Workflows)

Modular runbooks that can be executed on demand:

| Skill | Trigger / Use Case | Actionable Protocol |
| :--- | :--- | :--- |
| [react-feature-scaffold](./skills/react-feature-scaffold/SKILL.md) | Creating a new business feature | Scaffolds `api/`, `components/`, `hooks/`, `types/`, and `index.ts` public boundary. |
| [react-component-designer](./skills/react-component-designer/SKILL.md) | Building complex UI components | Guides decision between compound components, slot patterns, and headless hooks. |
| [react-performance-audit](./skills/react-performance-audit/SKILL.md) | Investigating lag or slow renders | 4-step diagnostic checklist to eliminate redundant renders and fix memory leaks. |
| [react-test-generator](./skills/react-test-generator/SKILL.md) | Writing unit/integration tests | Enforces AAA structure and accessibility-first query selection hierarchy. |

---

## 3. Agents (Specialist Personas)

Dedicated agent system prompts ready to be deployed as AI pair programmers or automated PR reviewers:

- [react-staff-architect.md](./agents/react-staff-architect.md): Senior tech lead guiding architecture, system design, and RFC decisions.
- [react-code-reviewer.md](./agents/react-code-reviewer.md): Automated PR reviewer enforcing zero anti-patterns, strict types, and clean code.

---

## 4. How to Install & Use in Any Project

### Method A: Workspace Customization (`.agents/`)
Copy the `agents/`, `rules/`, and `skills/` folders directly into the `.agents/` folder at the root of any target React project:
```bash
my-react-app/
└── .agents/
    ├── agents/
    ├── rules/
    └── skills/
```

### Method B: Global AI Configuration
Copy into your global assistant config (e.g., `~/.gemini/config/` or IDE configuration) to apply these rules universally across all projects.
