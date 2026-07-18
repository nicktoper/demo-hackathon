---
slug: surprise-lunch/optimize-a-surprise-meal-for-two
title: Optimize a surprise meal for two
status: draft
owner: nicktoper
human: nicktoper
agent: codex
assignee: nicktoper
contexts:
  - surprise-lunch/spec
skills: []
workflow: null
secrets: null
script: null
---

## Description

Build the selection engine that turns normalized DoorDash offers and menus
into qualifying carts for two people, obtains reversible checkout quotes, and
chooses a cheap surprise with controlled randomness. The output is both a
machine-readable selection and a redacted explanation of prices, promotion
application, and rejected candidates.

## Context

Depends on: `surprise-lunch/define-cli-and-configuration-contract` and
`surprise-lunch/discover-doordash-promotions-and-restaurants`.

The optimizer must compare verified checkout totals, not advertised discounts
or menu subtotals. Extend the provider only as needed to build and clear a cart
and retrieve a pre-submit quote; this ticket may make reversible cart changes
but may not activate the final order control. All food sufficiency, dietary,
budget, delivery-window, and controlled-randomness rules come from
`surprise-lunch/spec` and local configuration.

## Acceptance Criteria

- A pure candidate generator rejects carts that do not feed two people or
  violate any configured food, merchant, timing, or promotion constraint.
- Quote totals include discounts, credits, taxes, all fees, and configured tip,
  and preserve enough detail to explain comparisons without leaking private
  data.
- Selection first finds the cheapest qualifying verified total, then randomly
  chooses only within the configured price distance from that total.
- Randomness is injectable or seedable in tests, while normal invocations do
  not become predictably identical.
- Fixture tests cover promotion thresholds, BOGO carts, unavailable items,
  fee-driven ranking reversals, ties, no qualifying carts, and seeded
  reproducibility.
- Live dry-run mode obtains current quotes and reports the chosen cart without
  submitting an order or leaving stale cart state behind.

## Out of Scope

Clicking the final purchase control or claiming a delivery succeeded.

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.
