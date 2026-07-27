---
slug: surprise-lunch/reset-the-cli-contract-ticket-to-implement-and-cle
title: Reset the CLI contract ticket to implement and clear the dead worktree
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
`surprise-lunch/define-cli-and-configuration-contract` is stranded and cannot
make progress from where it sits.

**Source:** Dream 2026-W31 knowledge-scan findings K6 and K13, independently
corroborated by the Phase 1 validate-drift pass
(`stuck-in-progress` (warn) — `in_progress`, idle 232.6h).

### State

The ticket is `in_progress` at step 3 (`open-pr`). Its blackboard `## Dev`
section records `branch: feat/surprise-lunch-cli` and
`worktree: /tmp/demo-hackathon-surprise-lunch-cli`, and its `## Implemented`
section claims commit `543f0f7` with 93 passing tests.

None of that is recoverable:

- `feat/surprise-lunch-cli` is 98 behind / **0 ahead** of `origin/main` — the
  branch has no commits of its own.
- The worktree directory is gone; `git worktree list` shows the registration as
  `prunable`.
- The `recurring/rebase-stale-worktrees` run searched `origin/main`, the branch
  tree, the dead worktree's reflog, and every `git fsck --dangling` commit. No
  `src/` anywhere. It concluded the implementation lived in an independent
  `/tmp` clone and is gone.

So `open-pr` will fail on every relaunch: there is nothing to open a PR from.

### Work

1. Rewind the ticket to `implement` (`coga bump` supports a human rewind; Dream
   must not do this itself, and the validate-drift skill correctly refused to).
2. Correct the blackboard — the `## Implemented` claim is false and will
   mislead the next agent.
3. `git worktree prune` to clear the dead registration. Until then
   `git worktree add` refuses the branch. Three recurring jobs each declined to
   do this, every one citing another job's ownership, so nobody owns it —
   that is why it is bundled here rather than left to a sweep.

### Note

Also confirm whether `feat/surprise-lunch-cli` should be deleted and recreated
rather than reused, given it carries no work.

## Context

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.
