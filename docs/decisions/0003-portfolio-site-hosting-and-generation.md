# ADR 0003: Hosting and generation for the portfolio site

Date: 2026-09-04
Status: accepted

## Context

The objective this repository now exists to build is a personal portfolio site
with a projects section and a posts section, published at a public URL, whose
content can be added or changed by a coding agent connected to this repository
in a single change.

Three constraints were fixed before this decision and are not reopened here:

1. **Hosting is GitHub Pages, deploy-from-branch, `main` and `/docs`.** Pages
   is the only hosting reachable without a third-party account, and
   deploy-from-branch is a repository setting rather than a workflow. That
   matters because agents in this repository may never touch
   `.github/workflows/`, so any hosting that needs a deploy workflow puts the
   deployment permanently out of reach of the thing being built.
2. **The site must work with JavaScript disabled.** A portfolio that renders
   itself in the browser is a portfolio that is blank to anything that does not
   run scripts, including most link previews and crawlers.
3. **Everything worth verifying is verified by `npm test` and the CI gate.** A
   check that only a human can run is not a gate, and this repository exists to
   test whether the agent loop produces work that does not need reading.

Those three together decide the architecture rather than leaving it open.
Deploy-from-branch serves the files that are committed. Working without
JavaScript rules out assembling pages in the browser. So the HTML that Pages
serves has to exist in the repository, which means it is generated and
committed, which means CI has to prove the committed copy is the one the
generator produces.

Jekyll, which Pages would otherwise run, is ruled out by the third constraint:
its build happens on GitHub's side after a merge, so a template error surfaces
as a failed deployment rather than a failed check, and nothing in this
repository's Node toolchain can run it in CI.

## Decision

Content lives in `content/`, as plain Markdown files with a small front matter
block: `content/posts/` and `content/projects/`, one file per item. That
directory is the source of truth and the only place content is edited.

A generator written in TypeScript under `src/` reads `content/` and writes a
complete static site into `docs/`: HTML, CSS, and any assets, with no runtime
framework and no client-side rendering. `npm run build` runs it. `docs/` also
carries a `.nojekyll` file, so Pages serves exactly those bytes rather than
running Jekyll over them.

That last file is a deliberate departure from what the factory provisions. A
new repository arrives with Jekyll left on, so that its ADRs and research
render as a browsable site from day one without anything being built. That is
the right default for a project that publishes no product of its own. This one
publishes a generated site, and a generated site wants its bytes served
untouched, so Jekyll goes off here and the process records under
`docs/decisions/`, `docs/research/`, and `docs/design/` are served as raw
Markdown instead. Nothing links to them from the site; they are records, not
pages.

The generated site is committed. `npm test` regenerates it into a temporary
directory and fails if the result differs from what is committed under `docs/`,
so stale output is a red check rather than a wrong page. The generator writes
only the paths it generates and never removes `docs/decisions/`,
`docs/research/`, or `docs/design/`, which are hand-written process records
that happen to share the published directory.

This supersedes ADR 0001, which chose React, Vite, and React Router for a
favorites feature that is no longer the objective. ADR 0002, the favorites
`localStorage` schema, is withdrawn for the same reason: both describe a
product this repository is no longer building.

Two decisions are deliberately left open, because they are the kind this
repository's standing rules require an ADR for and the agent loop should be the
thing that makes them: the exact front matter and content schema, and how
Markdown becomes HTML - a dependency or a hand-rolled subset. Each needs its
own ADR in the same diff as the code that introduces it.

## Consequences

- Makes easy: adding a post or a project is writing one Markdown file and
  running one command. That is a workflow a coding agent can carry out in a
  single change, which is the objective's central requirement, and it is
  checkable by a test rather than by opinion.
- Makes easy: the served page is a build artifact of code under test. CI proves
  the exact bytes Pages will serve, so the only thing left for a human to check
  is whether it looks right.
- Makes hard: generated HTML in the diff. A content change shows up twice, once
  in `content/` and once in `docs/`. Review `content/` and `src/`; `docs/` is
  derived, and the regeneration test is what makes it safe to skip.
- Makes hard: forgetting to rebuild. The regeneration test turns that into a
  failing check rather than a silently stale site, but it is the mistake this
  layout invites and it will happen.
- Rules out later: nothing structural. Moving to a deploy workflow later would
  delete the committed output and the regeneration test, and would need a
  maintainer to add the workflow, since agents cannot.

## Alternatives rejected

- **React, Vite, and React Router, per ADR 0001.** Rejected because the
  objective changed out from under it. A bundler also puts the served page
  behind a build step whose output either gets committed anyway or needs a
  deploy workflow, and a client-rendered site fails the JavaScript-disabled
  constraint.
- **Jekyll, using Pages' own build.** The most content-friendly option, and it
  needs no committed output. Rejected because its build runs after merge on
  GitHub's side: a broken template is a failed deployment rather than a failed
  check, and this repository's gate cannot run Ruby.
- **Client-side rendering from Markdown or JSON fetched at runtime.** Needs no
  committed HTML and no build. Rejected because the site is then blank without
  JavaScript, and because what CI could test is the render functions rather
  than the page that actually gets served.
- **A deploy workflow building the site in Actions.** Keeps generated output
  out of the repository entirely. Rejected because agents here may never touch
  `.github/workflows/`, so the deployment would sit permanently outside the
  reach of the loop this repository exists to test.
