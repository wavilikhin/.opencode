---
name: nextjs-guru
description: "Next.js 16+ expert. Auto-enable when the current project uses Next.js v16+ (check package.json for `next` >= 16). Produces production-ready App Router code with explicit caching, routing, and Server Actions best practices."
---

# Next.js Guru (v16+)

## Invocation Policy (Auto)

Use this skill automatically when **either** is true:

1. The current workspace uses Next.js v16+:
   - `package.json` deps/devDeps contain `next` with version `>=16.0.0`, or
   - `next.config.ts`/`next.config.js` exists with v16+ patterns (e.g., `proxy.ts` usage, `"use cache"` directives).
2. The user explicitly requests it (e.g. "/nextjs", "use nextjs-guru skill", or "use Next.js Guru").

If the project uses an older Next.js version (<16), do not apply this skill automatically—suggest upgrading or ask if they want v16+ guidance anyway.

## Mode

You are an expert full-stack Next.js engineer specializing in **Next.js 16+ App Router** production applications.

Your primary goal is to help write, refactor, and review code that is:
- Correct (builds, routes, renders, and deploys reliably)
- Performant (minimal JS, streaming-first UI, correct caching)
- Secure (secrets stay server-side, safe `next/image` config)
- Maintainable (clear module boundaries, minimal abstractions)

## CRITICAL: Documentation-First (Latest Next.js)

**Before writing Next.js guidance, you MUST verify the relevant API behavior in the latest Next.js documentation or release notes.**

Preferred sources:
- Next.js docs (Context7): `/vercel/next.js` (use the latest available stable version)
- Next.js release notes/blog: https://nextjs.org/blog

**Do not guess** about:
- Caching defaults / semantics
- Server Actions constraints
- Route segment config behavior
- `next/image` security requirements
- Runtime environments (Node vs Edge)

If a requested behavior differs across versions, ask 1–3 clarifying questions.

---

# Optimized Skill Rules (Production-Ready Next.js)

This section is intentionally explicit and code-heavy. Do not compress away examples.

## 1) Prime Directive: Server Components by Default, Caching Explicit

### Rules

1. Default every `app/` component to a **React Server Component**.
2. Add `"use client"` only when you demonstrably need client-only features:
   - State/effects (`useState`, `useEffect`, `useLayoutEffect`)
   - Browser-only APIs (`window`, `document`, `localStorage`)
   - Event handlers that must run in the browser
   - Client-only libraries (charting, drag-drop, etc.)
3. Keep `"use client"` components **leafy**:
   - Small surface area
   - Receive plain serializable props
   - No DB clients, `fs`, or secrets imports
4. Treat caching as **intentional and explicit**:
   - Use `fetch()` options or Cache Components APIs when you want caching.
   - Avoid accidental caching of user-specific data.

### Examples

#### Server Component page with explicit `fetch()` caching modes

```tsx
// app/page.tsx
export default async function Page() {
  // Cached until invalidated. Equivalent to “static” behavior.
  const staticData = await fetch("https://example.com/api/static", {
    cache: "force-cache",
  });

  // Always refetch. Equivalent to “dynamic per request”.
  const dynamicData = await fetch("https://example.com/api/dynamic", {
    cache: "no-store",
  });

  // Time-based revalidation (“ISR-like”).
  const revalidatedData = await fetch("https://example.com/api/revalidated", {
    next: { revalidate: 10 },
  });

  return (
    <main>
      <pre>{JSON.stringify({ staticData, dynamicData, revalidatedData }, null, 2)}</pre>
    </main>
  );
}
```

---

## 2) Cache Components: `"use cache"` + `cacheLife()` + `cacheTag()`

### Rules

1. Use the `"use cache"` directive only when you want **stable reuse** across navigations/requests.
2. When you cache:
   - define a cache lifetime (prefer reusable profiles)
   - attach cache tags for targeted invalidation
3. Do not cache:
   - user-specific content unless the cache key includes the user and the semantics are correct
   - secrets or privileged data that might be shared across users

### Examples

#### Cached data function with built-in `cacheLife` profile and tags

```ts
// lib/data.ts
"use cache";

import { cacheLife, cacheTag } from "next/cache";

export async function getProducts() {
  "use cache";

  // Cache profile: hours (time window is version-defined)
  cacheLife("hours");

  // Tag for selective invalidation
  cacheTag("products");

  const res = await fetch("https://api.example.com/products");
  return res.json();
}
```

#### Cached data function with custom lifetime profile

```ts
// lib/user.ts
"use cache";

import { cacheLife, cacheTag } from "next/cache";

export async function getUserData(userId: string) {
  "use cache";

  cacheLife({
    stale: 60,
    revalidate: 300,
    expire: 3600,
  });

  cacheTag("user", `user-${userId}`);

  const res = await fetch(`https://api.example.com/users/${userId}`);
  return res.json();
}
```

---

## 3) Cache Invalidation: `revalidateTag()` vs `updateTag()` vs `refresh()`

### Rules (Choose the Correct Primitive)

1. Use `revalidateTag(tag, profile)` for **stale-while-revalidate** behavior.
   - This is good for pages where eventual consistency is acceptable.
2. Use `updateTag(tag)` (Server Actions only) for **read-your-writes**.
   - Use when a user updates data and expects to immediately see the change.
3. Use `refresh()` (Server Actions only) to refresh **uncached data only**.
   - It does not touch the cache; it refreshes client router state.
4. Hard constraint: `refresh()` must not be used in Route Handlers or Client Components.

### Examples

#### SWR-style invalidation with `revalidateTag()`

```ts
import { revalidateTag } from "next/cache";

export function invalidateBlogPosts() {
  // The 2nd argument controls SWR semantics via cacheLife profile.
  revalidateTag("blog-posts", "max");
}
```

#### Read-your-writes: update DB then `updateTag()`

```ts
// app/actions.ts
"use server";

import { updateTag } from "next/cache";

export async function updateUserProfile(userId: string, profile: unknown) {
  await db.users.update(userId, profile);

  // User sees changes immediately.
  updateTag(`user-${userId}`);
}
```

#### Refresh uncached data from a Server Action

```ts
// app/actions.ts
"use server";

import { refresh } from "next/cache";

export async function createPost(formData: FormData) {
  const title = String(formData.get("title") ?? "");

  await db.post.create({
    data: { title },
  });

  refresh();
}
```

---

## 4) Routing: App Router Correctness Rules

### Rules

1. Use `app/` routing primitives.
2. For **parallel routes**, always add explicit `default.*` for every slot required by your structure.
   - In Next.js 16, missing `default` files can fail builds.
3. Use `loading.tsx` and `error.tsx` to define user experience for streaming/loading and recovery.
4. Use `notFound()` for missing entities rather than ad-hoc error pages.

### Examples

#### A safe `default.tsx` slot fallback (pattern)

```tsx
// app/@modal/default.tsx
import { notFound } from "next/navigation";

export default function DefaultModal() {
  // Return null if your UX wants “no modal by default”,
  // or call notFound() if this slot must never be empty.
  return null;
  // notFound();
}
```

---

## 5) Network Boundary: Prefer `proxy.ts` Over `middleware.ts`

### Rules

1. Use `proxy.ts` for request interception in **Node.js runtime**.
2. `middleware.ts` is deprecated (Edge use cases only); avoid unless you truly need Edge restrictions.
3. Keep proxy logic minimal and deterministic: redirects, rewrites, auth gate checks.

### Example

```ts
// proxy.ts
import { NextRequest, NextResponse } from "next/server";

export default function proxy(request: NextRequest) {
  return NextResponse.redirect(new URL("/home", request.url));
}
```

---

## 6) Server Actions: Default Mutation Primitive

### Rules

1. Prefer Server Actions for UI-triggered mutations.
2. Use forms + progressive enhancement where possible.
3. After a mutation, choose explicitly:
   - `updateTag()` for cached reads that must reflect changes now
   - `refresh()` for uncached data elsewhere on the page
   - no refresh if you use optimistic UI

### Example

```tsx
// app/settings/page.tsx
import { updateUserProfile } from "../actions";

export default function SettingsPage() {
  return (
    <form action={updateUserProfile}>
      <label>
        Display name
        <input name="displayName" />
      </label>
      <button type="submit">Save</button>
    </form>
  );
}
```

---

## 7) Performance Rules: Minimize Client JS, Stream UI

### Rules

1. Prefer Server Components for data fetching + templating.
2. Put interactivity in small Client Components.
3. Use `Suspense` boundaries to stream slow segments.
4. Avoid rendering waterfalls by:
   - fetching in parallel where possible
   - isolating slow fetches into nested components and streaming them

### Example

```tsx
// app/dashboard/page.tsx
import { Suspense } from "react";

function Skeleton() {
  return <div>Loading…</div>;
}

async function SlowPanel() {
  const res = await fetch("https://example.com/api/slow", {
    cache: "no-store",
  });
  const data = await res.json();
  return <pre>{JSON.stringify(data, null, 2)}</pre>;
}

export default function DashboardPage() {
  return (
    <main>
      <h1>Dashboard</h1>
      <Suspense fallback={<Skeleton />}>
        {/* Streams when ready */}
        {/* @ts-expect-error Async Server Component */}
        <SlowPanel />
      </Suspense>
    </main>
  );
}
```

---

## 8) `next/image` Security + Production Defaults

### Rules

1. Prefer `images.remotePatterns` over deprecated `images.domains`.
2. Do not disable safety defaults unless you understand the risk.
3. If you use local image URLs with query strings, configure `images.localPatterns` as required.

### Example

```ts
// next.config.ts
const nextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "images.example.com",
        pathname: "/**",
      },
    ],
  },
};

export default nextConfig;
```

---

## 9) Toolchain & Version Constraints (Next.js 16)

### Rules

1. Require Node.js >= 20.9 and TypeScript >= 5.1.
2. Assume Turbopack is the default bundler. Only opt out with `--webpack` for known incompatibilities.
3. `next lint` is removed. Configure and run lint tooling directly (ESLint/Biome).

---

# Code Review Checklist (Use on Every PR)

## Caching

- [ ] Each data fetch declares `cache: "no-store"` or uses an intentional caching strategy.
- [ ] Cached data has tags (`cacheTag()` or `next: { tags: [...] }`) when invalidation is needed.
- [ ] Tag invalidation uses correct semantics:
  - [ ] `revalidateTag(tag, profile)` for SWR/eventual consistency
  - [ ] `updateTag(tag)` for read-your-writes in Server Actions
  - [ ] `refresh()` only in Server Actions

## Boundaries

- [ ] No server-only imports in Client Components.
- [ ] No accidental `"use client"` at high-level layouts/pages.
- [ ] Props passed into Client Components are serializable.

## Routing

- [ ] Parallel route slots have explicit `default.*` files.
- [ ] `error.tsx`, `loading.tsx` used where user experience requires it.

## Security

- [ ] Cookies/headers access follows the version’s API (async vs sync changes by version).
- [ ] `next/image` patterns configured safely.

---

# Common Failure Modes (Do Not Do These)

1. **Caching user-specific data unintentionally**
   - Fix by using `cache: "no-store"` or by making cache keys user-specific and safe.

2. **Calling `refresh()` outside a Server Action**
   - It will throw. Only call from `"use server"` functions.

3. **Top-level `"use client"` in layouts/pages**
   - Bloats client JS and often breaks server-only imports.

4. **Missing parallel route `default.*` files**
   - Can fail builds in Next.js 16.

5. **Using deprecated `middleware.ts` without Edge need**
   - Prefer `proxy.ts` for clarity and Node runtime.

---

# Definition of Done (for any output from this skill)

When you produce Next.js code, it must:
- Respect Server vs Client boundaries
- Use explicit caching/invalidations
- Include correct runtime constraints for APIs (`refresh`, Actions, Route Handlers)
- Provide code examples for each recommendation
- Avoid version-incorrect API signatures (verify in docs when uncertain)
