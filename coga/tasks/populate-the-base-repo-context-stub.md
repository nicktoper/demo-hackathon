---
slug: populate-the-base-repo-context-stub
title: Populate the base repo context stub
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

**Source:** Dream 2026-W30 knowledge-scan gap finding G2 (minor / low value in a
demo repo — safe to close).

`coga/context.md` is still the stock stub ("Describe what this repo is, who
works on it…") yet it is composed into every launched prompt. The repo's actual
identity — a coga demo whose live deliverable is the `surprise-lunch` DoorDash
CLI, agent `codex`, owner `nicktoper` — lives only scattered across tickets,
with no durable base-context carrier.

**Decision to make:** populate `coga/context.md` with the repo's real identity
and defaults, or leave the stub as-is for the demo.

## Context

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.
