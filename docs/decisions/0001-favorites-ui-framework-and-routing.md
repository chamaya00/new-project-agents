# ADR 0001: UI framework and routing approach for the favorites page

Date: 2026-09-03
Status: accepted

## Context

The favorites objective (#3) needs a favorites page that is separate from the
main list and reachable through navigation, with state that survives a page
refresh. `CLAUDE.md` states "Framework, data store, and hosting are not
chosen yet. Name each one here in the same commit that introduces it," so no
scaffold code for the favorites feature can start until a framework and
routing approach are on record. The repository currently has a bare
TypeScript-on-Node-20 scaffold (`src/index.ts`, `tsc`, `eslint`, `vitest`) and
no UI library, bundler, or router of any kind.

This issue's job is to name exactly one UI framework and one routing
approach so that #3's children C (framework scaffold) and D (favorites
store) have a contract to build against. It does not add the dependency —
that lands with the scaffold in C.

## Decision

Use **React** (18+) built with **Vite**, and **React Router** (v7, declarative
mode) for client-side routing between two routes: `/` (the main list) and
`/favorites` (the favorites page). Both routes render inside one
single-page app produced by the Vite build — there is no server-side
rendering or route-based data loading in scope here, since neither a data
store nor a hosting target has been chosen yet.

## Consequences

- Makes easy: React + Vite + React Router is the most-documented combination
  available, which lowers the odds of an engineer agent needing more than
  one attempt to implement C and D correctly. Vitest — already installed —
  is built by the same team as Vite and integrates with React through
  `@testing-library/react` with no separate transform configuration
  [Vitest in 2026](https://dev.to/ottoaria/vitest-in-2026-the-testing-framework-that-makes-you-actually-want-to-write-tests-kap).
  Vite also gives the repo a real `dev` script to add, which `CLAUDE.md`
  currently lists as missing.
- Makes hard: the bundle cost is disproportionate to what this app does.
  React Router v7 alone is roughly 58 KB min+gz
  ([Pi Stack, 2026](https://www.pistack.xyz/posts/2026-08-13-react-router-vs-tanstack-router-vs-wouter-typescript-comparison/)),
  and a React app with routing, state, and utilities commonly lands around
  200KB+ before feature code
  ([Should I Use This Framework?, 2026](https://shouldiusethisframework.com/blog/react-bundle-size-problem))
  — heavy for a two-page list-and-toggle app.
- Rules out later: nothing structural. If the data-store decision ever moves
  favorites off `localStorage` and onto a network source, React Router's
  data APIs leave room to add loaders without a routing rewrite. Switching
  UI frameworks later would still mean rewriting every component.

## Alternatives rejected

- **Preact + wouter**: React-compatible API at roughly 3 KB (Preact) + 2.2 KB
  (wouter) gzipped
  ([Pi Stack, 2026](https://www.pistack.xyz/posts/2026-08-13-react-router-vs-tanstack-router-vs-wouter-typescript-comparison/)).
  Rejected because the bundle savings only matter at a scale this sandbox
  app will not reach, while Preact's smaller ecosystem gives an
  implementing agent fewer canonical patterns to match against, raising the
  chance of a first attempt going wrong.
- **Svelte / SvelteKit**: compiles away the framework at build time for the
  smallest runtime cost of the options considered
  ([Strapi, "Svelte vs React in 2026"](https://strapi.io/blog/svelte-vs-react-comparison)).
  Rejected because it is a different component model and toolchain from the
  TypeScript/Vitest scaffold already in place, and would need its own
  test-runner integration instead of reusing Vitest directly.
- **No framework, hand-rolled routing**: rejected outright because the issue
  requires naming a UI framework, and a hand-written router plus DOM
  diffing for two pages is exactly the kind of undocumented ad hoc code an
  ADR is meant to prevent.
- **TanStack Router**: strongest type safety for route params and search
  state, and pairs well with TanStack Query
  ([Better Stack, "TanStack Router vs React Router"](https://betterstack.com/community/guides/scaling-nodejs/tanstack-router-vs-react-router/)).
  Rejected because that payoff scales with the size of the route tree; this
  app has exactly two routes, so the extra typed-route ceremony is not
  worth a less familiar API surface for a first implementation.
