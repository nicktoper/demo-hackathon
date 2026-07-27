---
schedule: "0 8 * * 1"
schedule_comment: "Every Monday at 8am — after branch-sweep deletes dead branches, rebase the live stale ones"
title: "Rebase stale worktrees"
# Runs as an agent: a rebase can hit conflicts, and deciding whether a
# textually-clean rebase is still semantically right needs judgment — the
# exact judgment the deterministic `coga open-pr` command refuses to fake.
# Launch on demand with `coga recurring launch rebase-stale-worktrees`
# whenever the open-pr staleness gate fires.
---

## Description

Bring live feature branches back up to date with the control branch so the
`code/open-pr` staleness gate passes on relaunch.

The `coga open-pr` command fails loud when a branch is materially stale relative
to `origin/main` — correct fail-loud behavior, because a stale PR can
reintroduce reverted work, but the remedy (rebase, re-verify, force-push) is
manual per branch. This task is that remedy, run over every live branch at
once. It is the counterpart to `branch-sweep`: branch-sweep deletes branches
whose work already landed; this task rebases branches whose work is still in
flight. Neither touches the other's set.

### Scope — what counts as live

1. Every non-main branch checked out in a worktree (`git worktree list`).
2. Every `branch:` recorded under a not-`done` ticket's `## Dev` section.

Skip everything else. A stale branch with no worktree and no live ticket is
abandoned or already-merged residue — branch-sweep's problem, not this task's.

### Run order

1. **Enumerate** — `git fetch origin main`, then for each live branch check
   `git merge-base --is-ancestor origin/main <branch>`. Collect the stale
   ones; record the up-to-date ones as no-ops.
2. **Rebase** — in the branch's own worktree when it has one, otherwise in a
   temporary worktree, and only from a clean tree: a dirty worktree is
   skipped and reported, never stashed. Then `git rebase origin/main`.
3. **Conflicts** — resolve only trivial mechanical conflicts (both sides
   appended to the same list, whitespace). Anything semantic:
   `git rebase --abort`, leave the worktree exactly as found, and report the
   branch with its conflicting files.
4. **Verify** — a textually clean rebase can still be semantically wrong
   (docs describing behavior main since changed, code building on a reverted
   commit). Re-read the branch's diff against the new base; run
   `python -m pytest` when it touches `src/` or `tests/`. Report — don't
   push — a branch whose content no longer holds.
5. **Push** — `git push --force-with-lease` only for branches that already
   have an upstream; an existing PR updates automatically. Never open a PR
   here — that belongs to each ticket's own `code/open-pr` step, which now
   passes its staleness gate on relaunch.
6. **Summarize** — replace the `## Rebase Run Summary` section in this
   blackboard with one line per branch: `rebased-pushed`, `rebased-local`,
   `up-to-date`, `conflict — human needed`, `skipped-dirty`, or
   `skipped-verify-failed`, plus the `coga launch <slug>` command for any
   ticket whose open-pr step is now unblocked. Replace, don't append — the
   blackboard is composed into every launch and must stay bounded.

### Safety rules

- Never delete a branch or worktree — branch-sweep owns deletion.
- Never touch `main`; never push without `--force-with-lease`.
- Never stash, commit, or reset a dirty worktree; skip and report it.
- A worktree already mid-rebase or mid-merge at the start of the run is
  someone's live session: report it, don't touch it.

<!-- coga:blackboard -->

`coga recurring` keeps the serviced-period high-water mark here as
`last_serviced_period`. Each run replaces the `## Rebase Run Summary`
section below with its results.

last_serviced_period: 2026-W31

## Rebase Run Summary

Run 2026-W31 (2026-07-27). Live set = 1 branch. Nothing rebased, nothing pushed.

- `feat/surprise-lunch-cli`: **skipped-verify-failed — human needed.** 98 behind
  `origin/main` and **0 commits of its own** (`git rev-list --left-right --count`
  = `98 0`; tip `f1ab386` is an ancestor of `origin/main`; branch reflog holds
  only `branch: Created from main`). Its registered worktree
  `/tmp/demo-hackathon-surprise-lunch-cli` no longer exists (`prunable`), so git
  refuses to check the branch out anywhere else — but rebasing is moot regardless:
  there is no work on it to rebase. The ticket's `## Implemented` section claims an
  installable `surprise-lunch` package; `src/` exists in neither `origin/main` nor
  the branch, the deleted worktree's own reflog records no commits, and
  `git fsck --dangling` turns up only `On main: autostash` blobs. The
  implementation was made in an independent `/tmp` clone and is **not recoverable
  from this repository.** Not fixable by this task — see Follow-ups.
- `dream/resync-phase4-retro-isolation`: out of scope — 28 behind / 1 ahead, but
  no worktree and no not-`done` ticket names it under `## Dev`. Same residue
  branch-sweep also left alone; a human owns it.

Corrects last run's entry, which reported `feat/surprise-lunch-cli` as
`rebased-local` with "2 own commits, 98 tests pass". No such commits exist on the
branch and none ever reached this repo's object store — that run was reporting on
an independent clone's state, not the branch's.

### Follow-ups (human)

1. `surprise-lunch/define-cli-and-configuration-contract` is `in_progress` at
   `open-pr` with nothing to open a PR from. It needs to go back to `implement`,
   not forward — do **not** relaunch it expecting the staleness gate to be the
   only problem.
2. Its `## Dev` `worktree:` line points at a deleted path and the branch is still
   registered to it, which is why the branch cannot be checked out. Clearing that
   registration is branch-sweep's / a human's call, not this task's.
