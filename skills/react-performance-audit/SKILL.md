---
name: react-performance-audit
description: >-
  Use this skill to audit React components and hooks for performance bottlenecks,
  unnecessary re-renders, state synchronization hazards, and incorrect useMemo/useCallback usages.
---

# Skill: React Performance Audit Workflow

This workflow provides a systematic profiling and remediation procedure for React rendering bottlenecks.

---

## 1. Audit Diagnostic Phase

When reviewing slow or laggy components, execute this 4-step check:

### Step 1: Detect State Duplication & Sync Hazards
Search for any `useEffect` that calls a `setState`:
- ⚠️ Is it updating state based on props? **Fix:** Replace with inline computed derived variable.
- ⚠️ Is it copying data from a query or fetch? **Fix:** Use data directly from the TanStack Query hook.

### Step 2: Identify Heavy Tree Rerenders
- Check React DevTools Profiler: Are components re-rendering because their parent re-rendered, despite their props being identical?
- **Action:** If the child tree is computationally heavy (> 50-100 DOM elements, Canvas, or deep lists), wrap the child in `React.memo`.

### Step 3: Audit Callback & Object Stability
- If a child is wrapped in `React.memo`:
  - Are you passing non-memoized inline objects (`style={{ width: 100 }}` or `config={{ id: 1 }}`)? **Fix:** Stabilize via `useMemo` or move outside component.
  - Are you passing inline arrow functions (`onClick={() => doSomething(id)}`)? **Fix:** Wrap callback in `useCallback`.

### Step 4: Purge Useless Memoization
- Search for `useMemo` wrapping primitive math or string operations.
- Search for `useCallback` passed directly to standard HTML tags (`<button onClick={...}>`).
- **Action:** Delete unnecessary `useMemo` / `useCallback` to reclaim CPU time and memory allocations.

---

## 2. Audit Report Format

When reporting findings, present:
1. **Identified Bottleneck:** Component name and line number.
2. **Root Cause:** State sync loop, unmemoized prop to `React.memo`, or expensive render computation.
3. **Proposed Refactoring:** Code diff with before/after.
