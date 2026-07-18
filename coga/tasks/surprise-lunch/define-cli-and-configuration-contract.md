---
slug: surprise-lunch/define-cli-and-configuration-contract
title: Define CLI and configuration contract
status: draft
owner: nicktoper
human: nicktoper
agent: claude
assignee: nicktoper
contexts:
  - surprise-lunch/spec
skills: []
workflow: null
secrets: null
script: null
---

## Description

Define and scaffold the local `surprise-lunch` command and the configuration
contract every later ticket will consume. The output is an installable command
skeleton with strict configuration validation, a dry-run surface, and no live
DoorDash behavior yet, so provider and checkout work can build on one stable
safety boundary.

## Context

This is the first project ticket and has no predecessor. No implementation
technology is fixed by the project interview; choose the smallest maintainable
stack that supports a packaged CLI, TOML configuration, deterministic tests,
and the browser provider planned by the next ticket, then record that choice in
the project README.

Use `surprise-lunch order` as the live command and
`surprise-lunch order --dry-run` as its non-purchasing counterpart. Store the
real configuration under the user's platform configuration directory (for
example `$XDG_CONFIG_HOME/surprise-lunch/config.toml`) and commit only a
placeholder-only example. Raw DoorDash credentials and payment-card data are
outside the configuration contract; follow `surprise-lunch/spec` for the
persistent-profile boundary.

## Acceptance Criteria

- The chosen project stack is initialized with an installable
  `surprise-lunch` executable and a documented local development command.
- One typed configuration model covers every required field in
  `surprise-lunch/spec`, and a placeholder-only example documents each field.
- Missing or invalid safety-critical configuration exits non-zero with a
  field-specific error before any provider code could run.
- `surprise-lunch order --dry-run` reaches a clearly marked provider stub and
  cannot place an order.
- Tests cover valid configuration, each required-field failure category, and
  redaction of sensitive values from errors and logs.

## Out of Scope

DoorDash discovery, meal selection, cart construction, and checkout.

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.
