---
title: agent-factory
summary: A reusable toolbox that turns high-level objectives into working code across projects.
url: https://github.com/chamaya00/agent-factory
date: 2026-08-15
---

Agent roles and process travel in a plugin, automation travels as reusable
workflows each project calls in one line, and everything a project learns stays
in that project.

The interesting constraint is that no agent may edit the gates it has to pass.
A system that can rewrite its own checks has no checks, so the workflows live
in one repository and every project calls them pinned to a release tag.
