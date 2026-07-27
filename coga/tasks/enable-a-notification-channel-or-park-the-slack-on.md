---
slug: enable-a-notification-channel-or-park-the-slack-on
title: Enable a notification channel or park the Slack-only recurring jobs
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
Two recurring jobs exist solely to post to Slack, and neither can post.

**Source:** Dream 2026-W31 knowledge-scan finding K10.

### State

`coga/coga.toml` has the fresh-repo default `[notification] channels = []`, and
`coga/coga.local.toml` contains only `user = "nicktoper"` — no override. So
every notification is suppressed to stderr.

Affected:

- `recurring/digest` — its whole output is "posts one sectioned message to the
  shared channel". Confirmed empirically by the 2026-07-27 run: 11 items
  rendered, spool drained, nothing delivered. "The daily digest is a no-op as a
  team signal until a channel is configured."
- `recurring/blocker-reminders` — "posts a live owner reminder".

Dream's own `coga slack` summaries are discarded the same way.

### Decision needed

Either:

- **Enable** — set `channels = ["slack"]` with an `env:` webhook reference
  (never a literal URL in the committed file), and consider
  `[notification.slack.users]` so owners get real pings; or
- **Park** — stop running two jobs whose entire output is discarded, and
  re-enable them when the team actually coordinates through Slack.

Note that the digest job still does useful non-Slack work: it drains the spool
and advances the git high-water mark. Parking it has side effects — check
before disabling rather than after.

## Context

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.
