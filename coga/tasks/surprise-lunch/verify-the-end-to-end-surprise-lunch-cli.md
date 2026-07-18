---
slug: surprise-lunch/verify-the-end-to-end-surprise-lunch-cli
title: Verify the end-to-end surprise lunch CLI
status: draft
owner: nicktoper
human: nicktoper
agent: codex
assignee: nicktoper
contexts:
  - surprise-lunch/spec
  - browser/dom-backed
  - autonomy/triage
skills: []
workflow: null
secrets: null
script: null
---

## Description

Turn the completed pieces into a documented, installable command and verify the
entire surprise-lunch path against fixtures, a live dry run, and one real order.
The project is done when the operator launches one non-interactive command for
a configured two-person lunch, receives an order ID and ETA, and the food
arrives at the configured WeWork location.

## Context

Depends on all preceding `surprise-lunch/` tickets, especially
`surprise-lunch/implement-guarded-automatic-checkout`.

The live demo is authorized by a complete local configuration rather than an
additional per-order approval prompt. It must use the ordinary production
selection, maximum-total, final-verification, and idempotency paths; no demo or
test flag may bypass them. If the account session or any required configuration
is missing, report the exact prerequisite and leave the live demo unrun.

The operator's physical receipt of the food is the final acceptance evidence.
Record the redacted order result in this ticket's blackboard and let the
operator add the delivery-arrived observation; do not store screenshots or
logs containing contact, address, payment, or session data.

## Acceptance Criteria

- The CLI has concise installation, profile-bootstrap, configuration,
  dry-run, live-order, failure-recovery, and uninstall instructions.
- The repository includes a safe example configuration and a fixture-backed
  test suite that runs without DoorDash credentials, network access, or a real
  purchase.
- A live dry run at the configured WeWork address demonstrates current offer
  discovery, two-person cart selection, verified pricing, and zero checkout
  submission.
- One ordinary `surprise-lunch order` invocation completes without an
  order-specific prompt and returns a confirmed order ID, total, restaurant,
  and ETA within every configured bound.
- The blackboard records the redacted command result and the operator confirms
  that the food arrived, satisfying the project demo defined in
  `surprise-lunch/spec`.

## Out of Scope

Supporting additional operators, delivery providers, offices, or a hosted
service before the single-user DoorDash demo works end to end.

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.
