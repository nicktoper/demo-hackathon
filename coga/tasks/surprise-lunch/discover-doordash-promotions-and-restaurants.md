---
slug: surprise-lunch/discover-doordash-promotions-and-restaurants
title: Discover DoorDash promotions and restaurants
status: draft
owner: nicktoper
human: nicktoper
agent: codex
assignee: nicktoper
contexts:
  - surprise-lunch/spec
  - browser/api-first
  - browser/dom-backed
skills: []
workflow: null
secrets: null
script: null
---

## Description

Implement the replaceable DoorDash provider that discovers promotions,
restaurants, and the menu data needed to build candidate lunches for the
configured account and delivery address. It should produce normalized,
testable data for the optimizer while keeping login bootstrap and all purchase
actions outside this ticket.

## Context

Depends on: `surprise-lunch/define-cli-and-configuration-contract`.

The API-first check performed during project planning found only partner- and
merchant-oriented DoorDash surfaces, not a generally available consumer API
for account-specific promotions and marketplace checkout. Recheck and record
the current official API coverage before implementing; use browser automation
only for the missing consumer workflow. DoorDash's current consumer terms also
make this an unofficial, fragile local integration, as recorded in
`surprise-lunch/spec`.

Use a dedicated persistent browser profile that the operator signs into
directly. Login, MFA, credentials, and payment data must never be collected by
the CLI. Discovery is read-only and DOM-backed; page structure or session
ambiguity must produce a diagnostic failure rather than partial or guessed
results.

## Acceptance Criteria

- A provider interface separates DoorDash-specific browser behavior from the
  CLI and optimizer domain models.
- The ticket records the official API endpoints or documentation checked and
  the exact consumer-discovery gaps that require browser automation.
- Normalized output captures each visible promotion's requirements and
  eligibility together with merchant identity, delivery estimate, and menu
  fields required to construct a two-person cart.
- Fixture-backed tests cover promotions including percentage-off, fixed
  discount, free item, BOGO, minimum subtotal, and ineligible or expired
  offers.
- A live read-only discovery path can use the configured address and persistent
  profile, redacts sensitive page data, and never enters or submits checkout.
- DOM drift, authentication expiry, bot challenges, missing offer terms, and
  incomplete extraction all fail loud with enough local diagnostics to repair
  the adapter.

## Out of Scope

Choosing a meal, mutating a live cart, or placing an order.

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.
