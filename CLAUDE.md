# Project context

## What this is

A personal portfolio site with a projects section and a posts section,
published by GitHub Pages at
[chamaya00.github.io/new-project-agents](https://chamaya00.github.io/new-project-agents/).

Adding or changing content is meant to be one file and one command, so that a
coding agent connected to this repository can do it in a single change. That is
the product requirement, not a convenience: if editing needs a CMS, an account,
or a database, the thing has been built wrong.

This repository doubles as the sandbox that exercises the agent-factory
workflow end to end. Both jobs point the same way - the site is what makes an
agent's work visible without reading the diff.

## Stack

TypeScript on Node 20. The Node version is pinned in
`.github/workflows/ci.yml`; changing one without the other means CI stops
testing what you run.

- **Hosting**: GitHub Pages, deploy from a branch, `main` and `/docs`. No
  deploy workflow, by design - see [ADR 0003](docs/decisions/0003-portfolio-site-hosting-and-generation.md).
- **Framework**: none, and that is a decision rather than a gap. The site is
  static HTML generated ahead of time and must work with JavaScript disabled.
- **Data store**: none. Content is Markdown files in `content/`.

Any of those three changing needs an ADR in `docs/decisions/` in the same
commit, and this section updated to match.

## Layout

- `content/posts/`, `content/projects/`, `content/about.md` - the content, one
  Markdown file per item, with front matter. **This is the only place content
  is edited.** It is seeded with two posts and two projects so that ordering
  and the multi-item cases are testable from the first run. The front matter
  those use is a starting shape, not the schema: the content-model ADR settles
  that, and migrating the seed files is part of the same change.
- `src/` - the generator, and its tests.
- `docs/` - the published site. **Generated. Never hand-edited.** Also holds
  `decisions/`, `research/`, and `design/`, which are hand-written process
  records the generator must leave alone.

When reviewing a change, read `content/` and `src/`. `docs/` is derived, and a
test proves it matches.

## Commands

- Install: `npm install`
- Dev: no dev script yet. Add `dev` to `package.json` and this line in the same
  commit, so the two never disagree.
- Typecheck / lint / test / build: `npm run typecheck`, `npm run lint`,
  `npm run test`, `npm run build`

The four scripts above are what CI runs. If one is renamed here, rename it in
`.github/workflows/ci.yml` in the same commit, or the gate silently stops
checking that thing. What a script does may change freely; what it is called
may not.

`npm run build` regenerates `docs/` from `content/`. Run it and commit the
result in the same change as the content, or `npm test` fails on stale output.

## Adding a post or a project

1. Write one Markdown file in `content/posts/` or `content/projects/`.
2. `npm run build`
3. Commit both the content file and the regenerated `docs/`.

If that ever takes more than those three steps, the site has drifted from what
it is for, and fixing it is a real issue rather than a chore.

## How work moves

Objectives become issues labelled `objective`. The orchestrator splits one into
2-5 child issues, each with acceptance criteria and one role label. A human
labels a child `agent:queued` when it is ready to run. Nothing runs itself.

Labels: `objective`, `agent:queued`, `agent:running`, `agent:review`,
`agent:blocked`, `needs-decomposition`, `role:researcher`, `role:designer`,
`role:engineer`.

## Standing rules

Acceptance criteria before work starts. Tests before merge. An ADR in
`docs/decisions/` for any schema or dependency change, in the same diff.

Everything worth verifying is verified by `npm test` or the CI gate. A
criterion that can only be checked by a person looking at the site is a
criterion that will pass by accident one day; the only thing left to human eyes
should be whether the page looks good.

Three failed attempts on one issue means the issue was scoped wrong. Stop and
ask for decomposition rather than trying a fourth time.

Never edit `.github/workflows/`, `CODEOWNERS`, or anything under a plugin
directory. If the work seems to need it, say so in a comment and stop.

## Memory

Lessons specific to this repository live in `.claude/memory/<role>.md`, one file
per role, 40 lines each. They are proposed in a pull request, never written
silently. A lesson that has graduated into a test, a lint rule, or a type gets
deleted - the check enforces it now, and the sentence is competing for attention
with the lessons nothing enforces yet.
