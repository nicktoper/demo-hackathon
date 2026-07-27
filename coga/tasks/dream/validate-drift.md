---
slug: dream/validate-drift
title: Validate drift
status: active
owner: nicktoper
human: nicktoper
agent: claude
assignee: claude
contexts: []
skills: []
workflow:
  name: dream/validate-drift
  steps:
  - name: run
    skills:
    - bootstrap/dream/tasks/validate-drift
    assignee: agent
secrets: null
script: null
step: 1 (run)
---

## Description



## Context

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.

## Dream Skill: validate-drift

Generated: 2026-07-27T22:42:11+00:00
Command: `/home/n/.local/share/uv/tools/coga/bin/python -m coga.validate --json --fix`
Task: `dream/validate-drift`

Result: 2 issue(s): 0 direct fix, 0 PR proposal, 2 human-needed.

### Human Needed

- `dream/validate-drift`: `broken-workflow` (error) - frozen workflow 'dream/validate-drift' cannot load its current definition: Workflow not found: /home/n/Code/demo-hackathon/coga/workflows/dream/validate-drift.md
  Remediation: Unknown validator issue kind. Ask a human before changing repo state.
- `surprise-lunch/define-cli-and-configuration-contract`: `stuck-in-progress` (warn) - in_progress but idle for 232.6h
  Remediation: Ask the owner whether the task should be relaunched, blocked, paused, or bumped. The skill should not change lifecycle state silently.

## Dream Skill: validate-drift

Generated: 2026-07-27T22:43:03+00:00
Command: `/home/n/.local/share/uv/tools/coga/bin/python -m coga.validate --json --fix`
Task: `dream/validate-drift`

Result: 1 issue(s): 0 direct fix, 0 PR proposal, 1 human-needed.

### Human Needed

- `surprise-lunch/define-cli-and-configuration-contract`: `stuck-in-progress` (warn) - in_progress but idle for 232.6h
  Remediation: Ask the owner whether the task should be relaunched, blocked, paused, or bumped. The skill should not change lifecycle state silently.
