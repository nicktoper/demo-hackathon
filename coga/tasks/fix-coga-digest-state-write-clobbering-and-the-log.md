---
slug: fix-coga-digest-state-write-clobbering-and-the-log
title: 'Fix coga digest state-write clobbering and the Log: commit filter'
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
Two defects in `coga digest`, both observed live in this repo. The fix lives in
the coga package (`/home/n/Code/claude/coga`), not in this repo — this ticket
tracks the decision and the upstream change.

**Source:** Dream 2026-W31 knowledge-scan findings K3 and K4.

### 1. The state write destroys neighbouring keys

`_STATE_RE` in `src/coga/commands/digest.py:52` matches from `### Digest State`
to the next heading *or end of file*:

```
rf"^###\s+{...Digest State}\s*$\n?(.*?)(?=^##\s|^###\s|\Z)"
```

`_write_digest_state` then splices `text[:match.start()] + state + text[match.end():]`.
Any key that sits after `### Digest State` with no intervening heading is inside
the match and is silently dropped — including `last_serviced_period`, which
`coga recurring` owns, not digest.

Observed impact: the key was clobbered and hand-restored on two consecutive
runs (see `coga/log.md`, 2026-07-27 15:35 "Now restoring the clobbered period
key in the parent"). If a run ever fails to restore it, the recurring scanner
sees no serviced mark for digest and can refire the same period, posting a
duplicate digest.

A repo-side mitigation shipped in PR #3 (moving `last_serviced_period` above
`### Digest State`), but that only hides the sharp edge. The real fix is that
the state write must preserve keys outside its own section.

### 2. `Log:` commits are not filtered out of "Also merged"

`src/coga/commands/digest.py:234-236` filters three subject prefixes:
`Sync task state:`, `Sync coga state`, and `Ticket: ... — ...`. It misses
`Log: <slug>`, emitted by `src/coga/recurring_runner.py:621`
(`git.sync_log(cfg, message=f"Log: {ref.id_slug}")`) on every period-task run.

So Coga's own bookkeeping commits get reported to humans as unattributed merged
work. The 2026-07-27 digest reported exactly four such commits under
"Also merged (no ticket)": `79ff79d`, `b903c42`, `fdc620e`, `ac5a8b4`.

### Scope note

Both fixes are in the coga package. Confirm with the owner whether this repo is
the right place to drive them, or whether they should be filed upstream.

## Context

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.
