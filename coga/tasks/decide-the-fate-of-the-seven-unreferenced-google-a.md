---
slug: decide-the-fate-of-the-seven-unreferenced-google-a
title: Decide the fate of the seven unreferenced Google ADK skills
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
Seven Google ADK skills sit under `coga/skills/google-agents-cli-*` (22 files)
in a repo whose only project is the `surprise-lunch` DoorDash CLI. They need a
keep-or-remove decision.

**Source:** Dream 2026-W31 knowledge-scan finding K7.

### Evidence

- Nothing references them. `grep -rn "google-agents" coga/tasks coga/recurring
  coga/workflows coga/contexts CLAUDE.md` returns only narrative mentions
  inside Dream/skill-update blackboards — zero `skills:` or `contexts:` entries.
- They carry no `name:` frontmatter, unlike every other skill and context in
  the repo (contrast `coga/skills/direct/body/SKILL.md` → `name: direct/body`).
- No `.coga-source.json` provenance exists anywhere in the repo.
- They arrived whole in `4dd99d1 Create coga via 'coga init'`.

### Cost of leaving them

The weekly `skill-update` job keeps generating PRs against them — open PR #2
touches 12 of these files. That is recurring review load for skills no ticket
uses.

### Decision needed

Either:

- **Keep** — add `name:` frontmatter so they resolve like every other skill,
  and create a ticket that actually uses them; or
- **Remove** — delete the tree and let `skill-update` stop tracking them.

### Constraint

Open PR #2 already edits these files. Do not open a competing PR — settle #2
first (merge or close), then act.

## Context

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.
