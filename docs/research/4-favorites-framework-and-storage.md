# Favorites persistence and page location — research for #4

Decision this serves: what UI framework and routing approach to build the
favorites page with, and what shape the `localStorage` data takes, so #3's
scaffold (child C) and store (child D) issues have a contract to build
against instead of each guessing separately.

## UI framework + routing

Two credible options for a small (two-route), client-only SPA:

### Option A — React + Vite + React Router

- What it does: component-based UI, Vite for dev server/build, React
  Router (declarative mode) for `/` and `/favorites`.
- What it costs: heaviest bundle of the options considered. React Router v7
  alone is ~58 KB min+gz; a typical React app with routing and utilities
  lands around 200KB+
  ([Pi Stack](https://www.pistack.xyz/posts/2026-08-13-react-router-vs-tanstack-router-vs-wouter-typescript-comparison/),
  [Should I Use This Framework?](https://shouldiusethisframework.com/blog/react-bundle-size-problem)).
- What it rules out later: nothing structural; React Router's data APIs
  leave room for server data loading if the store decision ever moves off
  `localStorage`.
- What would have to be true for it to be right: implementation reliability
  and ecosystem familiarity matter more than bundle size for this app —
  true here, since this is a sandbox exercising the agent workflow, not a
  performance-sensitive product.

### Option B — Preact + Vite + wouter

- What it does: same component model as React (via `preact/compat`), with
  a 2.2 KB router.
- What it costs: ~3 KB (Preact) + ~2.2 KB (wouter) gzipped
  ([Pi Stack](https://www.pistack.xyz/posts/2026-08-13-react-router-vs-tanstack-router-vs-wouter-typescript-comparison/)) —
  roughly 40x smaller than Option A.
- What it rules out later: nothing structural, but a smaller ecosystem
  means less canonical prior art to draw from when implementing.
- What would have to be true for it to be right: bundle size is a hard
  constraint for this app — not stated anywhere in #3 or `CLAUDE.md`.

**Inference** (not verified against this project directly): an implementing
agent is more likely to get a first attempt right with React + React Router
than with Preact + wouter, purely because there is more documentation and
prior art for the former. This is not measured for this repository — it's
a generalization from the libraries' relative popularity.

Also considered and rejected without a full writeup: Svelte/SvelteKit
(smallest runtime, but a different toolchain than the TS/Vitest scaffold
already in place) and TanStack Router (stronger typed routing, not worth it
for a two-route app). See ADR 0001 for the full comparison.

**Recorded in [ADR 0001](../decisions/0001-favorites-ui-framework-and-routing.md):
React + Vite + React Router.** Strongest argument against: the bundle is
disproportionately large for what the app actually does.

## localStorage schema

Two credible shapes for storing an unbounded list of favorites in
`localStorage`:

### Option A — single key, one JSON blob

- What it does: one `localStorage.getItem("favorites")` returns the whole
  list as one JSON document with a `version` field.
- What it costs: any corruption or quota failure affects all favorites at
  once, not one at a time.
- What it rules out later: per-entry partial writes; every update rewrites
  the whole blob.

### Option B — one key per favorite (`favorite:<id>`)

- What it does: each favorite is its own key/value pair.
- What it costs: listing all favorites requires scanning every
  `localStorage` key and filtering by prefix — there's no native "list keys
  matching a pattern" API, only `Object.keys(localStorage)` plus a filter.
  Versioning would need to live on every key instead of once.
- What it rules out later: nothing extra, but the query pattern the
  favorites page needs (read the whole list) is exactly what Option B makes
  awkward.

Regardless of shape, the failure modes are the same and are well
documented: `setItem` throws `QuotaExceededError` /
`NS_ERROR_DOM_QUOTA_REACHED` when storage is full
([Matteo Mazzarolo](https://mmazzarolo.com/blog/2022-06-25-local-storage-status/)),
`JSON.parse` throws `SyntaxError` on a corrupted or foreign value
([Chris Berkhout](https://chrisberkhout.com/blog/localstorage-errors/)), and
`localStorage` itself can be unavailable entirely in some browser
configurations (private browsing, disabled storage). All three need
explicit, separate handling — verified as the consistent recommendation
across multiple sources, not just one.

**Recorded in [ADR 0002](../decisions/0002-favorites-localstorage-schema.md):
single key `"favorites"`, versioned JSON blob, with named behaviour for
disabled/full/corrupt storage.** Strongest argument against: because it's
one blob, a single corruption event loses every favorite at once rather
than degrading gracefully.

## Sources

- [Pi Stack — React Router vs TanStack Router vs Wouter in 2026](https://www.pistack.xyz/posts/2026-08-13-react-router-vs-tanstack-router-vs-wouter-typescript-comparison/)
- [Should I Use This Framework? — React's Bundle Size Problem in 2026](https://shouldiusethisframework.com/blog/react-bundle-size-problem)
- [Strapi — Svelte vs React in 2026](https://strapi.io/blog/svelte-vs-react-comparison)
- [Better Stack — TanStack Router vs React Router](https://betterstack.com/community/guides/scaling-nodejs/tanstack-router-vs-react-router/)
- [Dev.to — Vitest in 2026](https://dev.to/ottoaria/vitest-in-2026-the-testing-framework-that-makes-you-actually-want-to-write-tests-kap)
- [Matteo Mazzarolo — Handling localStorage errors](https://mmazzarolo.com/blog/2022-06-25-local-storage-status/)
- [Chris Berkhout — Dealing with localStorage errors](https://chrisberkhout.com/blog/localstorage-errors/)
