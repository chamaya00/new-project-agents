# Project context

## What this is

A sandbox project used to exercise the agent-factory workflow end to end, for
the maintainer validating that provisioning, agent runs, and the merge gates
behave as intended.

## Stack

TypeScript on Node 20. The Node version is pinned in
`.github/workflows/ci.yml`; changing one without the other means CI stops
testing what you run.

Framework, data store, and hosting are not chosen yet. Name each one here in
the same commit that introduces it. The data store also needs an ADR in
`docs/decisions/`, because a store is a schema decision and the standing rule
below covers it.

## Commands

- Install: `npm install`
- Dev: no dev script yet. Add `dev` to `package.json` and this line in the
  same commit, so the two never disagree.
- Typecheck / lint / test / build: `npm run typecheck`, `npm run lint`, `npm run test`, `npm run build`

The four scripts above are what CI runs. If one is renamed here, rename it in
`.github/workflows/ci.yml` in the same commit, or the gate silently stops
checking that thing.

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
