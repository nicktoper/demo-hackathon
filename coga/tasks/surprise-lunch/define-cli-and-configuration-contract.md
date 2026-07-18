---
slug: surprise-lunch/define-cli-and-configuration-contract
title: Define CLI and configuration contract
status: active
owner: nicktoper
human: nicktoper
agent: codex
assignee: codex
contexts:
- surprise-lunch/spec
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

Define and scaffold the local `surprise-lunch` command and the configuration
contract every later ticket will consume. The output is an installable command
skeleton written in Python, with strict configuration validation, a dry-run
surface, and no live DoorDash behavior yet. Provider and checkout work should
be able to build on this ticket's stable CLI and safety boundary without
redefining configuration or command behavior.

## Context

This is the first project ticket and has no predecessor. Initialize a Python
package using `pyproject.toml` and the smallest maintainable dependency set that
supports an installed console command, TOML configuration, deterministic tests,
and the Playwright-based browser provider planned by the next ticket. Record the
chosen Python version, packaging commands, and test commands in the project
README.

Use `surprise-lunch order` as the live command and
`surprise-lunch order --dry-run` as its non-purchasing counterpart. Store the
real configuration under the user's platform configuration directory (for
example `$XDG_CONFIG_HOME/surprise-lunch/config.toml`) and commit only a
placeholder-only `config.example.toml` beside the Python CLI entry module. The
example must be safe to commit and must not be accepted as a live-ready
configuration merely because every TOML key is present. Raw DoorDash
credentials and payment-card data are outside the configuration contract;
follow `surprise-lunch/spec` for the persistent-profile boundary.

Both command forms parse configuration before entering a provider boundary and
then apply the mode-specific validation below. No setting or test hook may turn
the stub into a purchase path.

## Configuration Contract

### Discovery

The command reads exactly one real runtime file, with no current-working-directory
or adjacent-example fallback:

- Linux: `${XDG_CONFIG_HOME:-$HOME/.config}/surprise-lunch/config.toml`
- macOS: `$HOME/Library/Application Support/surprise-lunch/config.toml`
- Windows: `%APPDATA%\surprise-lunch\config.toml`

Use a standard platform-directory helper or equivalent tested logic. The config
loader may accept an injected path as an internal testing seam, but this ticket
does not add a public `--config` override. Ship the adjacent
`config.example.toml` as package data and document how to copy it to the real
location; runtime discovery must never load it implicitly.

### Typed TOML shape

The model may use nested Python types of the implementer's choice, but it must
expose these stable TOML keys and mode requirements:

| Key | Type | Required for | Contract |
| --- | --- | --- | --- |
| `version` | integer | all modes | Must be `1`; unknown versions fail. |
| `diners` | integer | all modes | Defaults to `2`; otherwise must be positive. |
| `delivery.address_line1`, `city`, `region`, `postal_code`, `country` | strings | dry and live | Non-empty delivery address; `address_line2` is optional. |
| `delivery.window_start`, `window_end` | `HH:MM` strings | dry and live | Same-day local window with end after start. |
| `delivery.timezone` | IANA timezone | dry and live | Must resolve through Python timezone data. |
| `delivery.dropoff_instructions` | string | live only | Explicit, non-placeholder value; no default. |
| `recipient.name`, `phone` | strings | live only | Explicit contact details; `email` is optional. |
| `pricing.currency` | string | dry and live | `USD` for this provider scope. |
| `pricing.max_checkout_total` | decimal string | dry and live | Positive hard ceiling, represented without binary floats. |
| `pricing.max_price_distance` | decimal string | dry and live | Non-negative absolute USD distance from the cheapest qualifying cart. |
| `tip.kind` | `percent` or `fixed` | dry and live | No default. |
| `tip.value` | decimal string | dry and live | Non-negative; percent is at most 100, fixed is in configured currency. |
| `food.dietary_requirements`, `allergens`, `excluded_ingredients` | string arrays | dry and live | Each key must be explicitly present, even when the intended value is empty. |
| `preferences.cuisines_allow`, `cuisines_deny`, `merchants_allow`, `merchants_deny`, `items_allow`, `items_deny` | string arrays | dry and live | Empty allow means unrestricted; any case-insensitive allow/deny intersection is invalid. |
| `provider.browser_profile_path` | path string | dry and live | Expand `~`, then require an absolute persistent-profile path. |
| `provider.expected_account` | string | live only | Account identity expected in the saved session; treat as sensitive in output. |
| `provider.saved_payment_method_label` | string | live only | Identifies an already-saved DoorDash method; never contains card credentials. |
| `ordering.default_dry_run` | boolean | all modes | Must be explicit in the real file; the example uses `true`. |
| `ordering.live_enabled` | boolean | all modes | Must be explicit in the real file; the example uses `false`. |

Decimal money strings permit at most two fractional digits and are parsed with
`Decimal`, never `float`. Trim strings before validation. Reject blank values,
example sentinels such as `<replace-me>`, invalid timezones/windows, malformed
paths, duplicate preference entries, and allow/deny conflicts with a
field-specific diagnostic.

Dry mode requires every field marked “all modes” or “dry and live.” Live-only
fields may be absent in dry mode, but any supplied value must still be
well-typed. Live mode requires the complete model. Passwords, MFA material,
session tokens, raw card numbers, and card security codes are invalid
configuration keys, not merely undocumented ones.

### Command behavior

Mode resolution is deterministic:

1. `order --dry-run` always selects dry mode.
2. Otherwise, `ordering.default_dry_run = true` keeps plain `order` in dry
   mode; setting it to `false` requests live mode.
3. A live request with `ordering.live_enabled = false` exits non-zero with
   “live ordering disabled.”
4. A fully configured and enabled live request in this ticket exits non-zero
   with “live provider not implemented,” before instantiating a provider.

The dry-run stub exits zero and writes a concise message to stdout identifying
dry-run mode, the absent provider implementation, and that no order was placed.
Usage, configuration, and live-mode failures write value-free diagnostics to
stderr and exit non-zero. Document and test the exact non-zero codes chosen by
the implementation.

Diagnostics may name a field path but must never echo field values, addresses,
contact details, account identity, browser paths, or payment labels. This
scaffold does not need persistent logging; if it introduces any logs, the same
redaction contract applies. Keep the provider seam injectable so tests can use
a recording double and prove that dry mode calls only the safe stub while live
refusals make no provider or purchase call.

## Acceptance Criteria

- A Python package declared by `pyproject.toml` installs a `surprise-lunch`
  console command, and the README documents installation, local development,
  configuration, and test commands.
- One typed configuration model covers every required field in
  `surprise-lunch/spec`, including explicit units or representations for money,
  time windows, timezones, and tip policy; the adjacent placeholder-only example
  documents every field without containing operator data.
- The runtime loader uses the documented platform configuration path, while the
  adjacent example is never silently treated as the real configuration.
- Missing or invalid safety-critical configuration exits non-zero with a
  field-specific, value-free error before any provider code could run, according
  to the documented dry/live validation split.
- `surprise-lunch order --dry-run` reaches only the clearly marked provider stub
  and exits zero without placing an order; plain `surprise-lunch order` follows
  the configured default-mode and live-enablement precedence before any
  provider-not-implemented refusal.
- Tests cover valid configuration, each required-field failure category, and
  redaction of sensitive values from errors and logs, plus proof that neither
  command can reach a purchase action through the injectable provider seam.

## Out of Scope

DoorDash public-API evaluation, terms rechecking, browser/profile bootstrap,
Playwright or DOM automation, promotion and restaurant discovery, meal
selection, cart construction, checkout, and any live network access. Those
provider-specific concerns begin with
`surprise-lunch/discover-doordash-promotions-and-restaurants`.

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.
