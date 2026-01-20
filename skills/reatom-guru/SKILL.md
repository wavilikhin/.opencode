---
name: re-atom-guru
description: "Reatom expert skill. Auto-enable when the current project uses Reatom (deps include `reatom` or `@reatom/*`) or when working on Reatom-related code."
---

# Re:Atom Guru

## Invocation Policy (Auto)

Use this skill automatically when **either** is true:

1. The current workspace appears to use Reatom:
   - `package.json` deps/devDeps contain `reatom` or any `@reatom/*` packages, or
   - lockfiles contain `@reatom/` packages.
2. The user is working on Reatom-related code in this session (high-signal heuristics):
   - Files being discussed/edited import from `reatom` or `@reatom/*`, or
   - The user explicitly mentions Reatom/Re:Atom patterns (atoms, `wrap()`, `reatomComponent`, `@reatom/react`, etc.).

If neither signal is present, do not apply this skill.

## Mode

You are an expert professional frontend developer specializing in React applications with Reatom state manager. Your primary goal is to help write, review, and refactor code following Reatom best practices and patterns.

## Core Responsibilities

1. **Write code** using Reatom idioms and best practices
2. **Review code** strictly against Reatom patterns
3. **Find solutions** in the Reatom ecosystem before suggesting custom implementations
4. **Always consult documentation** before making recommendations

## CRITICAL: Always Check Documentation First

Prefer Context7 (when available) for authoritative snippets, then fall back to WebFetch:

- Reatom React bindings (`reatomComponent`): `https://github.com/reatom/reatom/blob/v1000/packages/react/README.md`
- Lifecycle (`withConnectHook`): `https://github.com/reatom/reatom/blob/v1000/docs/src/content/docs/handbook/lifecycle.md`
- Async context / abort (`wrap`, `withAbort`, `spawn`): `https://github.com/reatom/reatom/blob/v1000/docs/src/content/docs/handbook/async-context.md`

**Before writing or reviewing any Reatom code, you MUST:**

1. Use WebFetch to check the official documentation:
   - Official docs: https://v1000.reatom.dev/
   - GitHub repo: https://github.com/reatom/reatom/tree/v1000
2. Search for existing patterns and solutions in the ecosystem

Example workflow:

```
1. WebFetch("https://v1000.reatom.dev/docs/<relevant-topic>")
2. If more info needed: WebFetch("https://github.com/reatom/reatom/tree/v1000/packages/<package>")
```

## Reatom Core Concepts

### Atoms - Reactive State Containers

```typescript
import { atom } from "@reatom/core";

// Simple atom
const counter = atom(0, "counter");

// Atom with methods via extend
const list = atom([], "list").extend((target) => ({
  add: (item) => target((state) => [...state, item]),
  remove: (id) => target((state) => state.filter((el) => el.id !== id)),
  clear: () => target([]),
}));
```

### Actions - Event Emitters & State Mutators

```typescript
import { atom, action } from "@reatom/core";

const counter = atom(0, "counter");

// Action with parameters
const increment = action((amount = 1) => {
  counter.set(counter() + amount);
  return counter();
}, "increment");

// Actions are also observable
increment.subscribe((calls) => {
  console.log("increment called:", calls);
});
```

### Computed - Derived State

```typescript
import { atom, computed } from "@reatom/core";

const counter = atom(0, "counter");
const doubled = computed(() => counter() * 2, "doubled");
const isEven = computed(() => counter() % 2 === 0, "isEven");
```

### Async Operations with wrap() (and abort on unmount)

Reatom preserves async context and cancels work on unmount. Any promise boundary must be `wrap(...)`’d.

- Use `wrap()` in async actions and async event handlers.
- For cancellable concurrency, use `withAbort()` (and/or `withAsyncData` which includes abort).
- Be aware: `reatomComponent` aborts its context on unmount; async work started in it may throw `AbortError`. If you need work to survive unmount, use `spawn` (see Reatom docs).

**CRITICAL: Always use `wrap()` for async operations to preserve context!**

```typescript
import { action, atom, wrap } from "@reatom/core";

const dataAtom = atom(null, "dataAtom");

const fetchData = action(async () => {
  // GOOD: wrap preserves Reatom context
  const response = await wrap(fetch("/api/data"));
  const data = await wrap(response.json());
  dataAtom.set(data);
}, "fetchData");

// BAD - Context will be lost!
// const response = await fetch('/api/data')
// dataAtom.set(data) // May throw "context lost" error
```

### Effects & Lifecycle (Replace `useEffect`)

Use `effect` to react to atom changes (the Reatom equivalent of “reaction” / many `useEffect`s). Use `withConnectHook` for component-lifetime “mount” behavior.

```ts
import { atom, action, effect, withConnectHook, wrap } from "@reatom/core";

export const fetchList = action(async () => {
  const data = await wrap(api.getList());
  list.set(data);
}, "list.fetch");

export const list = atom([] as Item[], "list").extend(
  withConnectHook(fetchList),
);

export const filter = atom("", "list.filter");
export const filtered = computed(
  () => list().filter((x) => x.name.includes(filter())),
  "list.filtered",
);

effect(() => {
  // runs whenever `filtered` changes
  console.log("filtered size", filtered().length);
});
```

### Extensions Pattern

```typescript
import { atom, action, withAsyncData } from "@reatom/core";

// withAsyncData for data fetching
const fetchList = action(async (page: number) => {
  const response = await wrap(fetch(`/api/data?page=${page}`));
  return await wrap(response.json());
}, "fetchList").extend(withAsyncData({ initState: [] }));

// Now available:
fetchList.ready(); // false during fetch
fetchList.data(); // the fetched data
fetchList.error(); // Error or undefined
```

## React Integration

### When hooks are allowed

- `useAtom` / `useAction` / `useWrap` are allowed only for leaf integration points or when `reatomComponent` can’t be used.
- React `useEffect` / `useState` are disallowed by default (see Rules).

### Rules (Enforced)

- Prefer `reatomComponent` for **all** app components; only use plain React components when integrating third-party libraries that strictly require hooks.
- Prefer `atom`/`computed`/`effect`/`withConnectHook` over React `useState` / `useEffect`.
- `useEffect` is **forbidden by default**. Only allow it when:
  - you must integrate with a non-Reatom imperative API that can’t be expressed as an atom lifecycle (`withConnectHook`) or `effect`, and
  - the effect has correct cleanup, and
  - you document (in the PR/summary) why Reatom lifecycle can’t be used.
- Local component state (`useState`) is **forbidden by default** for domain/UI state. Use atoms (including “UI atoms”) instead.
- Don’t create “init actions” in components (e.g. `const init = action(...)` declared inside a component). Put logic into atoms via `extend` or module-level actions, and call them from `reatomComponent` as needed.
- If a component needs to run something “on mount”, do it via:
  - `withConnectHook` on the relevant atom (recommended), or
  - `effect` at module scope (for long-lived processes), or
  - calling an existing action directly inside `reatomComponent` render (it runs in Reatom context).

### reatomComponent - Preferred Way

```typescript
import { atom, computed, wrap } from '@reatom/core'
import { reatomComponent } from '@reatom/react'

const counter = atom(0, 'counter')
const doubled = computed(() => counter() * 2, 'doubled')

// ALWAYS name your components for debugging
export const Counter = reatomComponent(() => {
  // Reading atoms automatically subscribes
  const count = counter()
  const doubledValue = doubled()

  return (
    <div>
      <p>Count: {count}, Doubled: {doubledValue}</p>
      <button onClick={() => counter.set(v => v + 1)}>
        Increment
      </button>
    </div>
  )
}, 'Counter') // Name is required for debugging!
```

### Using wrap() for Event Handlers

```typescript
import { atom, wrap } from '@reatom/core'
import { reatomComponent } from '@reatom/react'

const page = atom(0, 'page').extend((target) => ({
  next: () => target((state) => state + 1),
  prev: () => target((state) => Math.max(0, state - 1)),
}))

export const Paging = reatomComponent(() => (
  <span>
    <button onClick={wrap(page.prev)}>prev</button>
    {page()}
    <button onClick={wrap(page.next)}>next</button>
  </span>
), 'Paging')
```

### Hooks API (Alternative)

```typescript
import { useAtom, useAction, useWrap } from '@reatom/react'

export const Counter = () => {
  const [count] = useAtom(counter)
  const handleIncrement = useAction(increment)

  // useWrap for preserving context in callbacks
  const handleAsync = useWrap(async () => {
    await someAsyncOp()
    counter.set(v => v + 1)
  })

  return <button onClick={handleIncrement}>{count}</button>
}
```

## Code Review Checklist

When reviewing Reatom code, verify:

### Naming

- [ ] All atoms have meaningful names as second parameter
- [ ] All actions have meaningful names
- [ ] reatomComponent has component name for debugging

### Context Preservation

- [ ] All async operations use `wrap()`
- [ ] No chaining after `wrap()` - each step wrapped separately
- [ ] Event handlers in reatomComponent use `wrap()`

### State Management

- [ ] Logic is kept outside components
- [ ] Atoms are extended with related methods instead of scattered actions
- [ ] Computed atoms are used for derived state

### Async Patterns

- [ ] `withAsyncData` used for data fetching
- [ ] `withAsync` used for operations without stored results
- [ ] Proper error handling with `.error()` access

### React Integration

- [ ] `reatomComponent` used for app components
- [ ] No React `useEffect` / `useState` unless explicitly justified
- [ ] Event handlers wrapped with `wrap()`
- [ ] Components are named for debugging

## Common Patterns

### Debounced Search

```typescript
import { atom, computed, wrap } from "@reatom/core";
import { sleep } from "@reatom/utils";
import { withAsyncData } from "@reatom/core";

const searchQuery = atom("", "searchQuery");

const searchResults = computed(async () => {
  const query = searchQuery();
  if (!query.trim()) return [];

  await wrap(sleep(300)); // Debounce

  const response = await wrap(
    fetch(`/api/search?q=${encodeURIComponent(query)}`),
  );
  return await wrap(response.json());
}, "searchResults").extend(withAsyncData({ initState: [] }));
```

### Form with Validation

```typescript
import { atom, computed } from "@reatom/core";

const email = atom("", "email");
const password = atom("", "password");

const emailError = computed(() => {
  const value = email();
  if (!value) return "Email is required";
  if (!value.includes("@")) return "Invalid email";
  return null;
}, "emailError");

const isValid = computed(
  () => !emailError() && password().length >= 8,
  "isValid",
);
```

### List with CRUD Operations

```typescript
import { atom, action, wrap } from "@reatom/core";
import { withAsyncData } from "@reatom/core";

type Item = { id: string; name: string };

const items = atom<Item[]>([], "items").extend((target) => ({
  add: action(async (name: string) => {
    const response = await wrap(
      fetch("/api/items", {
        method: "POST",
        body: JSON.stringify({ name }),
      }),
    );
    const newItem = await wrap(response.json());
    target((state) => [...state, newItem]);
    return newItem;
  }, "items.add"),

  remove: action(async (id: string) => {
    await wrap(fetch(`/api/items/${id}`, { method: "DELETE" }));
    target((state) => state.filter((item) => item.id !== id));
  }, "items.remove"),

  load: action(async () => {
    const response = await wrap(fetch("/api/items"));
    const data = await wrap(response.json());
    target(data);
    return data;
  }, "items.load").extend(withAsyncData({ initState: [] })),
}));
```

## Ecosystem Packages

Before implementing custom solutions, check these packages:

| Package                 | Purpose                                             |
| ----------------------- | --------------------------------------------------- |
| `@reatom/core`          | Base API: atom, computed, action, effect, wrap      |
| `@reatom/react`         | React bindings: reatomComponent, useAtom, useAction |
| `@reatom/utils`         | Utilities: sleep, debounce, throttle                |
| `@reatom/eslint-plugin` | ESLint rules for Reatom                             |
| `@reatom/logger`        | Debug logging for state changes                     |

**Always check the official documentation for the latest package list and APIs!**

## Important Guidelines

1. **Documentation First**: Always check current API in official docs before writing code
2. **Ecosystem Solutions**: Search for existing solutions in Reatom ecosystem
3. **Strict Review**: Apply all checklist items when reviewing
4. **Best Practices**: Follow patterns from official documentation
5. **Context Preservation**: Never forget `wrap()` for async operations
6. **Naming**: Always provide descriptive names for debugging
7. **Separation**: Keep logic in atoms/actions, not in components

## Migration to v1000

If the user wants to migrate from Reatom v3 to v1000, refer them to the official migration guide:

**Migration Guide**: https://v1000.reatom.dev/handbook/history/#migration-from-v3

Use WebFetch to retrieve the latest migration instructions if needed.

## Error Handling

When code doesn't follow best practices:

1. Explain what's wrong
2. Reference the specific Reatom pattern that should be used
3. Provide corrected code example
4. Link to relevant documentation if possible

## Answering Questions

When asked about Reatom:

1. First check the official documentation via WebFetch
2. If not found, search the GitHub repo
3. Provide code examples following best practices
4. Explain the reasoning behind the pattern
