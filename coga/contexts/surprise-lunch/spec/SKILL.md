---
name: surprise-lunch/spec
description: Product, configuration, selection, and checkout contract for the personal surprise-lunch CLI.
---

# Surprise Lunch Specification

## Outcome

`surprise-lunch order` is a local command the operator runs whenever they want
lunch. One invocation selects a good-value DoorDash meal for two, places one
order to the configured delivery location, and reports the resulting order ID,
total, restaurant, and ETA. The end-to-end demo is successful when the command
runs without an order-specific prompt and the food arrives.

This is a personal, single-operator tool. It is not a multi-user service, a
promotion-resale product, or a way to create accounts, stack ineligible offers,
or otherwise manipulate DoorDash promotions.

## Surprise-mode contract

- There is no per-order menu or checkout approval. The operator authorizes the
  behavior once through local configuration.
- The command chooses with controlled randomness among qualifying carts close
  to the cheapest verified checkout total, rather than always ordering the
  absolute cheapest item or ignoring price in the name of surprise.
- A qualifying cart feeds two people and satisfies every configured dietary,
  cuisine, item, merchant, delivery-time, and price constraint.
- Promotion percentages and menu subtotals are not final prices. Candidate
  comparison uses the checkout total after discounts, taxes, service and
  delivery fees, tip, credits, and any small-order or distance charges.
- Only promotions shown as eligible for the configured account, address, cart,
  and current order may be used. If eligibility cannot be verified at checkout,
  the candidate is rejected.

## Local configuration and secrets

The implementation owns one documented, gitignored local configuration file.
It must include, at minimum:

- `diners` (default `2`)
- delivery address, recipient contact details, and drop-off instructions
- delivery window and timezone
- a hard maximum checkout total
- tip policy
- dietary requirements and exclusions
- cuisine or merchant allow/deny preferences
- the permitted price distance from the cheapest qualifying cart
- browser-profile location and the saved payment-method selector or label
- live-ordering enablement and dry-run defaults

Real ordering must fail loud until every safety-critical field has been set.
There are no guessed defaults for maximum spend, allergies, tip, recipient
contact, or delivery instructions.

DoorDash passwords, MFA material, raw card numbers, and card security codes do
not belong in the configuration file, repository, logs, or command line. The
tool uses a dedicated persistent browser profile that the operator signs into
directly. Payment remains stored by DoorDash; configuration may identify an
already-saved payment method without containing its credentials. An expired
session is a hard stop that requires the operator to refresh the profile before
a later invocation.

## Provider boundary

All DoorDash-specific behavior lives behind one replaceable provider adapter.
The public developer surface must be checked first and the ticket must record
which parts, if any, it covers. UI automation is limited to consumer promotion
discovery and checkout behavior that the public API does not provide. Start
with DoorDash's Marketplace integration and Checkout API documentation:

- https://developer.doordash.com/en-US/docs/marketplace/how_to/order_integration/
- https://developer.doordash.com/en-US/api/external_checkout/

DoorDash's consumer terms currently restrict automated collection, scraping,
and use of DoorDash content in software without permission. Personal use does
not make the integration official or stable. The project must keep live access
local, make the provider risk explicit, provide fixture-backed development and
dry-run paths, and avoid representing the tool as approved by DoorDash. Recheck
the current terms before enabling live access:
https://help.doordash.com/en-us/legal/article/cx-terms-and-conditions.

Browser interaction is DOM-backed: use fresh Playwright accessibility
snapshots and element refs or locators. Do not target screenshots, use OCR, or
guess coordinates. If the required state or control cannot be established from
the DOM, fail loud rather than selecting a nearby element.

## Automatic-checkout invariants

Immediately before the irreversible submit action, the command re-verifies:

1. the persistent session belongs to the configured account;
2. the delivery address and recipient match configuration;
3. the cart is sufficient for two and satisfies all food constraints;
4. the selected promotion is applied and its requirements remain satisfied;
5. the final total is at or below the configured hard maximum;
6. the tip and delivery window match configuration; and
7. no live order from this invocation has already been submitted.

The command uses an invocation lock and a durable submission record. If submit
returns an unknown result, it checks order history before retrying and otherwise
stops; it never blindly repeats an uncertain purchase. A concurrent invocation
or an existing unresolved submission is a hard failure.

## Output and evidence

Dry-run output explains the qualifying candidates, applied promotions, final
totals, rejections, and chosen cart without placing an order. Live output adds
the DoorDash order ID, restaurant, charged total, and ETA. Logs are local,
bounded, and redact credentials, payment details, contact information, session
tokens, and unnecessary page content.

Tests use stored fixtures and mocked checkout transitions by default. A live
test must use the same configured maximum, idempotency guard, and final
verification path as a normal invocation; test-only code may not bypass them.
