---
slug: surprise-lunch/implement-guarded-automatic-checkout
title: Implement guarded automatic checkout
status: draft
owner: nicktoper
human: nicktoper
agent: claude
assignee: nicktoper
contexts:
  - surprise-lunch/spec
  - browser/api-first
  - browser/dom-backed
  - autonomy/triage
skills: []
workflow: null
secrets: null
script: null
---

## Description

Connect the selected cart to a fully automatic DoorDash checkout that places
exactly one order without asking for per-order approval. The irreversible
action is allowed only after the command re-verifies every configured bound and
can prove that the invocation has not already submitted or entered an unknown
submission state.

## Context

Depends on: `surprise-lunch/define-cli-and-configuration-contract`,
`surprise-lunch/discover-doordash-promotions-and-restaurants`, and
`surprise-lunch/optimize-a-surprise-meal-for-two`.

The human explicitly chose surprise mode: a correctly configured live
invocation must not introduce a restaurant, item, or checkout confirmation
prompt. Bound the financial and food risk through the one-time configuration
and the pre-submit invariants in `surprise-lunch/spec`; if any invariant is
unknown or false, failing without an order is the intended behavior.

Persist locks and submission state under the user's platform state directory,
separate from configuration and browser data. Treat an ambiguous submit result
as unresolved until order history establishes whether DoorDash created an
order; never retry the final action merely because the browser timed out.

## Acceptance Criteria

- Live checkout is unreachable unless local configuration explicitly enables
  it and all safety-critical fields validate.
- Immediately before submission, the implementation verifies the configured
  account, address, recipient, two-person cart, dietary constraints, applied
  promotion, final total, tip, delivery window, and payment-method selector.
- A single-instance lock prevents concurrent orders, and durable invocation
  state prevents duplicate submission across crashes and restarts.
- Unknown submit outcomes reconcile against order history and otherwise stop
  for recovery without clicking again.
- Mocked browser tests cover success, every hard stop, concurrent invocation,
  price or item changes at checkout, session expiry, pre-click failure,
  post-click timeout, and an already-created order.
- On confirmed success the command prints and records a redacted order ID,
  merchant, charged total, and ETA, then releases only the locks that are safe
  to release.

## Out of Scope

Bypassing MFA, creating or rotating DoorDash accounts, stacking ineligible
promotions, handling refunds, or automatically retrying a failed delivery.

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.
