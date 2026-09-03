# ADR 0002: Favorites localStorage schema

Date: 2026-09-03
Status: accepted

## Context

The favorites objective (#3) already fixed the persistence mechanism: "No
accounts - local storage is fine." Both the main list page (to know which
items are already favorited) and the favorites page (to render the saved
list) need to read the same data, and it must survive a page refresh. The
standing rule in `CLAUDE.md` requires an ADR for any schema decision, and
`localStorage` itself is unreliable in ways the schema has to plan for: it
can be disabled (private browsing, browser settings), it can be full
(`setItem` throws `QuotaExceededError` / `NS_ERROR_DOM_QUOTA_REACHED`
([Matteo Mazzarolo, "Handling localStorage errors"](https://mmazzarolo.com/blog/2022-06-25-local-storage-status/))),
and a stored value can be unparseable JSON if it was ever written by
something else or corrupted
([Chris Berkhout, "Dealing with localStorage errors"](https://chrisberkhout.com/blog/localstorage-errors/)).
This issue records the schema; it does not write the store implementation
(that is #3's child D).

## Decision

Store all favorites under a single `localStorage` key: `"favorites"`. The
value is a JSON string with this shape:

```json
{
  "version": 1,
  "items": [
    { "id": "string", "addedAt": "2026-09-03T18:00:00.000Z" }
  ]
}
```

- `version` (number): the schema version of this payload, starting at `1`.
  Bumped whenever `items`' shape changes.
- `items` (array): one entry per favorited item.
  - `id` (string): the identifier of the favorited item in whatever list
    data source is eventually chosen. This ADR does not define what an
    `id` refers to beyond "the source list's own identifier" — that
    depends on the still-undecided data store.
  - `addedAt` (string): ISO-8601 timestamp of when the item was favorited,
    used to order the favorites page by recency.

Required behaviour for the three failure cases:

1. **localStorage disabled or inaccessible** (access throws, or
   `window.localStorage` is undefined): treat every read as an empty list
   and every write as a no-op beyond updating in-memory state for the
   current page load. The UI shows no error — this is expected browser
   behaviour, not a fault.
2. **localStorage full** (`setItem` throws `QuotaExceededError` or
   `NS_ERROR_DOM_QUOTA_REACHED`): retry the write exactly once. If it still
   fails, keep the last known in-memory state (do not lose favorites
   already rendered this session) and surface a non-blocking inline notice
   that the favorite could not be saved. Do not crash the page.
3. **Stored value is unparseable JSON, or parses but `version`/`items` is
   missing or malformed**: treat it as if no favorites exist — start from
   an empty list — and overwrite the corrupted value the next time a write
   succeeds. Do not attempt partial recovery of individual entries.

A stored `version` lower than the code's current version runs a migration
before use. A stored `version` higher than the code's current version is
treated the same as case 3 (unparseable): fall back to an empty list rather
than guessing at an unknown shape.

## Consequences

- Makes easy: a single key means one `getItem`/`setItem` pair to reason
  about and test; the `version` field lets a future change to `items`
  migrate existing users instead of silently discarding their data.
- Makes hard: because all favorites live in one JSON blob, a quota or
  corruption problem affects the whole list at once rather than degrading
  one entry at a time.
- Rules out later: nothing structural, but this ADR does not address
  syncing favorites across open tabs (a `storage` event listener would be
  needed for that) — deferred, not precluded.

## Alternatives rejected

- **One `localStorage` key per favorite** (e.g. `favorite:<id>`): rejected
  because listing all favorites would require scanning every key in
  `localStorage` and filtering by prefix, which is fragile, and each key
  would need its own `version` field instead of one shared value.
- **IndexedDB instead of localStorage**: rejected because the parent
  objective already fixed `localStorage` as the persistence mechanism; this
  issue's job is the schema within that constraint, not re-opening it.
- **No `version` field, defensive parsing only**: rejected because the
  issue's acceptance criteria require a schema version field, and without
  one a future shape change would be indistinguishable from corrupted data.
