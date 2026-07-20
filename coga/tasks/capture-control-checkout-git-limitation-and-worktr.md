---
slug: capture-control-checkout-git-limitation-and-worktr
title: Capture control-checkout git limitation and worktree recovery as durable context
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

**Source:** Dream 2026-W30 knowledge-scan gap finding G1 (low confidence — a
"should we?" for the owner; safe to close if judged as environmental noise).

The repo has repeatedly hit control-checkout git failures with no durable
carrier to warn the next `code/*` run:

- `surprise-lunch/define-cli-and-configuration-contract` recorded, at its
  `open-pr` step, read-only git metadata in the registered worktree
  (`index.lock` unwritable), a broken `codex` sandbox launcher
  (`bwrap: execvp … codex: No such file or directory`), and GitHub DNS
  failures — plus an ad-hoc human-approved recovery: replace the read-only
  registered-worktree metadata with an independent writable `/tmp` clone.
- The (now retired) `recurring/rebase-stale-worktrees` run independently
  reported `feat/surprise-lunch-cli` stuck local-only (only `origin/main`
  exists on the remote).

**Decision to make:** is this durable enough to warrant a context/skill (e.g.
`coga/contexts/dev/control-checkout`) documenting the limitation + the
worktree/independent-clone recovery, or are the root causes purely
environmental (sandbox mount, launcher path, DNS) and best left uncaptured?

Note: the Dream contract-audit drift PR (resync Dream Phase 4 to packaged
isolated-checkout Retro guidance) partly overlaps this territory; consider
that PR's outcome before adding a new carrier.

## Context

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.
