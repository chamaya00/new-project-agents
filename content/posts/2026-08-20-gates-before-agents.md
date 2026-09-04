---
title: Gates before agents
date: 2026-08-20
summary: Prove the check refuses something before you point an agent at it.
---

The temptation is to wire up the agents first and add the checks once something
is working. That order produces work you have to read line by line, which is the
cost the agents were supposed to remove.

A gate that has never refused anything is not known to be a gate. Break a test
on purpose, watch the merge button go grey, and only then let anything
automated near the repository.
