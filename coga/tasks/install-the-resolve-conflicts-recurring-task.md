---
slug: install-the-resolve-conflicts-recurring-task
title: Install the resolve-conflicts recurring task
status: draft
owner: nicktoper
human: nicktoper
agent: claude
assignee: claude
contexts: []
skills: []
workflow:
  name: code/with-review
  steps:
  - name: implement
    skills:
    - code/implement
    assignee: agent
  - name: peer-review
    skills: []
    assignee: other-agent
  - name: open-pr
    skills:
    - code/open-pr
    assignee: agent
    requires: pr
  - name: review
    skills: []
    assignee: owner
secrets: null
script: null
step: 1 (implement)
---

## Description
Coga ships a `resolve-conflicts` recurring template that this repo does not
have.

**Source:** Dream 2026-W31 contract-audit finding C8.

### Why it is missing

`coga init` seeds recurring templates once, at init time, and never back-fills.
`resolve-conflicts` was added upstream after this repo was initialized, so the
packaged-vs-repo diff reports `Only in .../recurring: resolve-conflicts`. The
`coga resolve-conflicts` CLI command and the `bootstrap/resolve-conflicts`
command ticket are both present in the live CLI.

### Why it may matter here

This repo has an unowned stale-branch backlog — `dream/resync-phase4-retro-isolation`
sat open and unreviewed for a full period, and `feat/surprise-lunch-cli` is 98
commits behind with a dead worktree. That is the backlog this template exists
to work.

### Decision needed

A human should decide whether this repo wants a weekly automatic rebase of open
PR heads with force-push. That is a real risk trade, not a mechanical install:

- It force-pushes branches that humans may have checked out.
- Its scope overlaps the repo-local `rebase-stale-worktrees` task, which
  already enumerates worktrees and ticket `## Dev` branches. Running both
  unreconciled means two jobs with different safety rules touching the same
  refs.

If adopted, resolve the overlap with `rebase-stale-worktrees` in the same
change — either narrow that task or retire it.

## Context

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.
