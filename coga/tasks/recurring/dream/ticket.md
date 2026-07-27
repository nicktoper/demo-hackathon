---
slug: recurring/dream
title: Dream
status: done
owner: nicktoper
human: nicktoper
agent: claude
assignee: claude
contexts:
- coga/period-task
skills: []
workflow:
  name: direct/body
  steps:
  - name: execute
    skills:
    - direct/body
    assignee: agent
secrets: null
script: null
---

## Description

Run the Dream cleanup pass for this Coga repo.

Dream is Coga's generic cleanup pass. It runs in two halves. The **decide**
half reads the whole repo while it is still intact and classifies every
housekeeping repair and knowledge change worth making. The **execute** half
turns those decisions into reviewable PRs, tracked draft tickets, and safe
repairs. Every Dream finding ends in a durable artifact — a PR, a draft
ticket, or a recorded marker — never only in this task's blackboard, which a
later Dream run retires along with the task.

Dream is not REM. Repo/user-specific recurring maintenance belongs in a
separate REM task under `coga/recurring/`, with its own cadence, skill
order, and output conventions.

### Console Progress

Write short progress updates to the console before and after each phase:
validate-drift, knowledge scan, contract audit, Retro pass,
cleanup-orphan-markers, disposition, and the final status mark. Include the
command or file path being
acted on and the result count when available. If a phase is skipped, say why.
The blackboard remains the durable record; console progress is for the human
watching the run.

### Run order

Dream runs six phases in order. Phases 1–3 **decide** — they read the repo and
record what to change. Phases 4–6 **execute** — they make the changes. Deciding
before executing is deliberate: the knowledge scan and contract audit read the
corpus while every done ticket still exists (Phase 4 deletes them all), so
nothing is missed, and their findings steer the Retro pass.

1. **validate-drift** — deterministic repo hygiene (script worker).
2. **knowledge scan** — one full-corpus read; classifies every finding.
3. **contract audit** — checks the contract surface against code reality.
4. **retro/done-ticket** — extracts durable knowledge from every eligible done
   ticket in one pass.
5. **cleanup-orphan-markers** — delete-only orphan cleanup (script worker).
6. **disposition + run summary** — routes every finding to a durable home.

This body is the dispatch contract. Do not auto-discover skills, scan a plugin
folder, or invent another maintenance phase during the run. Adding or removing
a Dream phase is a normal change to this template. A phase failing does not
permit a replacement: record the result and continue only with later phases
whose inputs do not depend on the blocked one. If a repo wants a different
maintenance loop, make another task with its own body and ordered phase list.

The two script workers (Phases 1 and 5) each run as a child script
task whose one workflow step references the worker skill — Dream-owned scripts
are skills attached to Coga tasks, never standalone execution units. Before
launching a worker, read its `## Known Skill Contract`, keep its reads and
writes inside its declared scope, let it write its own `## Dream Skill: <name>`
section to the child task blackboard, then summarize that child result here.

### Phase 1 — validate-drift

Launch a child script task whose current workflow step references
`bootstrap/dream/tasks/validate-drift`. The skill runs the same deterministic
surface as `coga validate --json`, classifies every issue, and appends
`## Dream Skill: validate-drift` to the child task's blackboard.

The skill's default safe-repair pass applies only deterministic repairs
currently supported by `coga validate --fix`: append a missing blackboard fence
+ rendered region to a `ticket.md` that lacks one. The single-file format keeps
state in `ticket.md`'s blackboard region — there is no sibling `blackboard.md`
or `log.md`, and append-only history goes to the repo-global `coga/log.md`. It
does not rewrite existing files, synthesize `ticket.md`, freeze workflows, or
change lifecycle/assignee state.

### Phase 2 — knowledge scan

Delegate this phase to a subagent using the
`bootstrap/dream/scan/knowledge-scan` skill. This decide-half scan happens
before Phase 4 so done-ticket evidence is still available.

Write the returned findings to this task's blackboard under `## Findings`;
Phase 4 reads that section when batching knowledge PRs.

### Phase 3 — contract audit

Delegate this phase to a subagent using the
`bootstrap/dream/scan/contract-audit` skill. This decide-half audit complements
Phase 1's deterministic repo-hygiene check.

Write the returned findings to this task's blackboard under `## Findings`,
alongside the Phase 2 findings; Phase 6 reads that section when routing
proposal PRs.

### Phase 4 — retro/done-ticket

Extract durable knowledge from done tickets, then delete every one of them.
This pass processes **every eligible done ticket in a single run** — there is
no per-run ticket cap and nothing is deferred to a later run. One corpus read
with one running delta across all tickets is both cheaper than repeated capped
runs and better at de-duplicating repeated facts.

A done ticket is eligible when:

- its resolved task directory under `coga/tasks/` still exists; and
- no open PR is adding its `## Retro` marker or deleting that resolved task
  directory.

A ticket whose directory is already gone is not a candidate; git history holds
its record. A processed `## Retro` marker on a still-present directory does not
settle the ticket — its deletion PR has not merged, so it stays eligible. Do
not infer completion from branch names, stale comments, or old Dream notes —
only the on-disk directory and open-PR state count.

Run `retro/done-ticket <slug> [<slug> ...]` in one subagent, passing every
eligible slug. The skill loads the context/skill corpus once, reads each
ticket, carries one running delta across the whole run, and partitions the
tickets into coherent PR batches — each PR within its hard limits (≤5 source
tickets, ≤3 knowledge files, ≤1 new context/skill file, one theme). Every
processed done ticket is deleted: a ticket that contributed durable knowledge
is deleted in its theme's knowledge PR, which also records its `## Retro`
marker; a ticket carrying nothing durable is direct-deleted with
`coga delete <slug>` (a working-tree `git rm` plus a direct
`Ticket: <slug> — deleted` commit), with no PR and no marker. Recovery is via
`git restore`. Retro never leaves a processed done ticket on disk and never
opens a marker-only PR.

A done `recurring/<name>` ticket from this sweep is eligible like any other.
Period tickets carry nothing durable (their output is the notification post or
PR they already produced), so Retro direct-deletes them via `coga delete
recurring/<name>` — no PR or marker — while leaving the recurring template's
`last_serviced_period` untouched. If a completed period ticket survives into a
later firing, the recurring scanner deletes it before creating that period's
fresh task. The previous Dream run is removed by that scanner fallback before
this Dream task is created, so Dream never sees or deletes its own predecessor.

Summarize each knowledge PR — and the directly-deleted no-knowledge tickets —
in this run's blackboard.

### Phase 5 — cleanup-orphan-markers

Recovery path for done tickets whose blackboard carries a processed Retro
marker from a knowledge PR but whose task directory was not deleted by that
PR. Phase 4 knowledge PRs delete the source directory in the same PR, so this
pass should usually find nothing. A no-durable-knowledge ticket is direct-deleted
by Phase 4 in the run and never carries a `## Retro` marker, so it can never be a
candidate here; the gate still excludes any `result: no-new-durable-knowledge`
marker left behind by an older run.

Launch a child script task whose current workflow step references
`bootstrap/dream/tasks/cleanup-orphan-markers`. The skill detects cleanup
candidates and gates deletion through `bootstrap/delete-task`. That delete
skill ships, but until its cleanup PR-dispatch wiring is finished the worker
reports `human-needed` and deletes nothing.

For each candidate, cleanup must open a PR that deletes only the resolved task
directory under `coga/tasks/`. The deletion goes in the PR, not the working
tree, so a human can review it before merge. Cleanup gate:

- the marker is present in the task directory's `ticket.md` blackboard region;
- the marker does not have `result: no-new-durable-knowledge`;
- no open PR is currently editing that task directory;
- the exact task slug is known; do not use prefix matching for deletion;
- the PR deletes only that resolved task directory;
- the PR body states that git history is the audit trail.

Result line: `pr-opened` when the PR is opened. If any gate is unclear, write
`human-needed` instead of opening the PR. Do not auto-merge.

### Phase 6 — disposition + run summary

Every Phase 2 and Phase 3 finding gets a durable home. The `## Findings`
blackboard section is an index of what Dream saw, not where decisions go to
rest — this task is retired and its blackboard with it.

Route each finding by class:

- `extract` — already handled by Phase 4 (a knowledge PR, or — when the ticket
  carried nothing durable — a direct `coga delete`).
- `stale` — open a proposal PR that edits the named context or skill to match
  reality. The PR is `pr-required`: a human reviews and merges it; Dream never
  auto-merges and never edits a context or skill directly on `main`. If a
  stale fix would touch a context or skill that a Phase 4 PR already edits, do
  not open a conflicting PR — note the overlap on the finding and leave it for
  that PR's review.
- `drift` — open a proposal PR that fixes the named contract: correct the doc
  to match code, repoint or remove a dead reference, or resync a diverged
  packaged/live copy pair. Like `stale`, the PR is `pr-required` and Dream
  never auto-merges. If the fix overlaps a context or skill a Phase 4
  knowledge PR already edits, note the overlap and defer to that PR's review.
- `gap` — create a tracked draft ticket with
  `coga create "<title>" --workflow code/with-review`. A gap needs human
  design judgment about whether and how to add the context, skill, or
  workflow; a draft ticket is where that judgment happens, and unlike a
  blackboard note it survives this task's retirement.

Then append one top-level `## Dream Run Summary` section to this task's
blackboard: the generation time, a phase result table using the vocabulary
`no-op`, `reported`, `proposed`, `direct-fixed`, `pr-opened`, `human-needed`,
the finding counts with one-line summaries, links to every PR opened and draft
ticket created, and any `human-needed` decisions or review gates. Keep it short
enough for a human to scan.

### Slack

Child script tasks write their durable result to their own blackboard; the
parent Dream run sends the broader one-line summary. Call:

`coga slack --task <this-dream-task> --message "<summary>"`

Keep the message to one line, for example:
`Dream: validate-drift clean, 2 knowledge PRs, 1 stale-fix PR, 1 gap ticket.`

Run `coga mark done <this-dream-task>` once the blackboard is up to date and
the Slack summary is posted. That is the last action — **do not delete this
task.** The run's durable artifacts — every PR, draft ticket, and the Slack
summary — carry the findings, so this `done` task and its blackboard are
disposable, but Dream does not delete itself mid-run. It sits on disk as a
done `recurring/dream` ticket; at the next firing, the recurring scanner deletes
that prior-period artifact and creates a fresh Dream task from this template.
Git history preserves the completed run.

## Context

<!-- coga:blackboard -->

The blackboard is a notepad to be written to often as the human and agent works through a task.

## Run notes

- Repo: `/home/n/Code/demo-hackathon`. Period `2026-W31`.
- Live CLI is an **editable install** at `/home/n/Code/claude/coga` (shadows the
  uv site-packages 0.3.0 copy). Packaged template resolution reads
  `/home/n/Code/claude/coga/src/coga/resources/templates/coga/bootstrap/`.
- Open PRs at run start: #1 (`dream/resync-phase4-retro-isolation`, edits
  `coga/recurring/dream/ticket.md`), #2 (`coga/skill-update`, edits 12 files
  under `coga/skills/google-agents-cli-*`). Both must be treated as overlap
  gates for Phase 6 proposal PRs.

### Phase 1 — validate-drift (`reported`)

Dispatch deviation, recorded as a drift finding: the live CLI no longer ships
`bootstrap/workflows/dream/*`, so a child script task could not be created from
a packaged workflow ref. The workers moved to registered recipes
(`coga run validate-drift` / `coga run cleanup-orphan-markers`) and the two
`bootstrap/dream/tasks/*` SKILL.md files dropped their `script: run.py`
frontmatter. Restored `coga/workflows/dream/{validate-drift,cleanup-orphan-markers}.md`
as repo-local overrides (commit `b19e5ba`) so the child tasks' frozen workflow
refs load, then executed via the recipe.

Child task: `dream/validate-drift`. Result: 1 issue — 0 direct-fix,
0 pr-proposal, 1 human-needed.

- `surprise-lunch/define-cli-and-configuration-contract`: `stuck-in-progress`
  (warn) — `in_progress`, idle 232.6h. Lifecycle decision belongs to the owner;
  the skill must not change it silently.

### Phase 4 eligibility (established while Phases 2–3 ran)

Six done tickets on disk, all `recurring/<name>` period tickets:
`autoclose-merged`, `blocker-reminders`, `branch-sweep`, `digest`,
`rebase-stale-worktrees`, `skill-update`. None carries a `## Retro` marker
(0 in every file). No open PR edits anything under `coga/tasks/` — PR #1 edits
`coga/recurring/dream/ticket.md`, PR #2 edits `coga/skills/google-agents-cli-*`.
So all six are eligible.

Caveat on the template's premise: the body asserts period tickets "carry
nothing durable". False for `recurring/digest` this sweep — its blackboard
records two concrete coga defects (digest state-write clobbers
`last_serviced_period`; the "Also merged (no ticket)" filter matches only
`Sync coga state`, not `Log: ...`). `recurring/branch-sweep` also records a
residual manual decision on branch `dream/resync-phase4-retro-isolation`.
Those must land in durable artifacts before the tickets are deleted.

### Dream template drift — full picture

`diff` of the live packaged template
(`/home/n/Code/claude/coga/src/coga/resources/templates/coga/recurring/dream/ticket.md`)
against `coga/recurring/dream/ticket.md` shows **two** divergences:

1. **Phases 1 & 5 dispatch** (run-order lines, the worker paragraph, Phase 1
   body, Phase 5 body): packaged says *registered recipe*, run
   `coga run validate-drift` / `coga run cleanup-orphan-markers` directly from
   the Dream task, inheriting `COGA_TASK_*`, and explicitly **"Do not create
   child script tasks."** Repo still says *script worker* + child script task.
   This is the drift that blocked Phase 1 of this run.
2. **Phase 4 isolation** (evidence snapshot, isolated checkout,
   `--keep-control-checkout`, cleanup/verification): packaged has it, repo
   does not. **Already covered by open PR #1**, whose premise I re-verified
   against the live package — still valid and current.

Overlap handling: both live in the same file, but in disjoint hunks. Phase 6
opens a complementary PR for (1) only, leaving PR #1 to carry (2).

## Findings

### Phase 2 — knowledge scan (`reported`, 15 findings)

Classes: 0 `extract`, 4 `drift`, 3 `stale`, 8 `gap`.

| id | class | target | one-line |
|---|---|---|---|
| K1 `dream-template-phase15-drift` | drift | `coga/recurring/dream/ticket.md` | Phases 1/5 still say "child script task"; live CLI uses registered recipes. Overlaps PR #1 (disjoint hunks). |
| K2 `recurring-templates-workflow-vs-recipe` | drift | `coga/recurring/{autoclose-merged,blocker-reminders,branch-sweep,digest,skill-update}/ticket.md` | All five frozen at 0.3.0 `workflow:` shape; packaged now uses `recipe:`. Digest also drifted on scope (Done vs Done/Canceled). |
| K3 `digest-clobbers-last-serviced-period` | drift | `coga/recurring/digest/ticket.md` | `_write_digest_state` regex swallows keys after `### Digest State`, dropping `last_serviced_period`. Hand-restored 2 runs running. |
| K4 `digest-log-commits-unfiltered` | drift | `coga/recurring/digest/ticket.md` | "Also merged" filter matches `Sync coga state` / `Ticket:` but not `Log: <slug>`. |
| K5 `dream-child-task-residue` | gap | `coga/tasks/dream/validate-drift.md` | This run's Phase 1 child task is `active`; no Dream phase reaps it. |
| K6 `surprise-lunch-ticket-stranded-at-open-pr` | gap | `coga/tasks/surprise-lunch/define-cli-and-configuration-contract.md` | `in_progress` at step 3 `open-pr` with a 0-commit branch + dead worktree; needs to go back to `implement`. Matches Phase 1's `stuck-in-progress`. |
| K7 `unreferenced-google-adk-skills` | gap | `coga/skills/google-agents-cli-*` | 7 skills, 22 files, zero refs anywhere; no `name:` frontmatter, no provenance. PR #2 overlap — do not open a competing PR. |
| K8 `claude-md-canonical-context-claim-false` | stale | `CLAUDE.md` + `AGENTS.md` | Claims canonical contexts are "composed into every launched ticket"; `compose.py` does no such injection — they're only on `bootstrap/orient`. Also says 3, package ships 7. |
| K9 `w30-gap-tickets-still-draft` | gap | `populate-the-base-repo-context-stub`, `capture-control-checkout-git-limitation-and-worktr` | W30 gap drafts untouched a week. **Do not re-create duplicates.** |
| K10 `notifications-disabled-recurring-jobs-are-noops` | stale | `coga/coga.toml` vs digest/blocker-reminders templates | `channels = []`, no local override — two Slack-only recurring jobs are silent no-ops. |
| K11 `skill-update-provenance-description-stale` | stale | `coga/recurring/skill-update/ticket.md` | Describes `.coga-source.json` provenance walk; no such file exists, real run used a `gh-managed` delegation path. |
| K12 `digest-owner-nick-mismatch` | stale | `coga/recurring/digest/ticket.md` | `owner: nick` vs `nicktoper` everywhere else; will mis-@ once Slack is on. |
| K13 `prunable-worktree-unowned` | gap | `/tmp/demo-hackathon-surprise-lunch-cli` | Prunable worktree registration blocks `git worktree add`; three jobs each deferred to another. Merge into K6. |
| K14 `stale-dream-resync-branch` | gap | PR #1 | Open + unreviewed a full period; every sweep re-reports it. |
| K15 `gitignore-orphan-comments` | stale | `coga/.gitignore` | Comments duplicated below the coga-managed end marker with no patterns. Cosmetic. |

**Phase 4 input:** no `extract` findings. All six done period tickets were read
in full and carry no durable knowledge — their content is either run transcript
or observations already captured above as K3/K4/K6/K11/K13. Direct-delete all
six, no PR, no marker.

**Phase 6 routing guidance from the scan:** three proposal PRs — (a) K1;
(b) K2+K11+K12 (+K3 repo-side mitigation); (c) K8 (+K15). Four draft tickets,
not eight — K3+K4 (upstream digest fixes), K6+K13 (surprise-lunch reset),
K7 (ADK disposition), K10 (Slack enablement). K5, K9, K14 are `human-needed`
summary gates, not new tickets.

### Phase 3 — contract audit (`reported`, 9 findings)

Classes: 6 `drift`, 2 `stale`, 1 `gap`. Plus one operator-side note.

| id | class | target | one-line | overlap |
|---|---|---|---|---|
| C1 `recurring-templates-missing-recipe-key` | drift | `coga/recurring/{autoclose-merged,blocker-reminders,branch-sweep,digest,skill-update}/ticket.md` | **Highest value.** Missing `recipe:` frontmatter → `recurring_runner.py:519` falls through to `launch_cmd`, spawning a full agent per job. `coga/log.md:162-185` confirms: branch-sweep 672k, digest 1187k, skill-update 653k cache-read tokens for work that should be agentless. | none |
| C2 `dream-template-child-script-task-model` | drift | `coga/recurring/dream/ticket.md` | Same as K1. Packaged says run recipes directly, "Do not create child script tasks". | **PR #1**, disjoint hunks — stack on its branch |
| C3 `repo-workflows-claim-script-step` | drift | `coga/workflows/{autoclose-merged/sweep,blocker-reminders/run,branch-sweep/sweep,skill-update/run}.md` | All four still say "Script step"; referenced skills dropped `script:`. No repo-local state — straight copy is safe. | none |
| C4 `vestigial-dream-workflow-overrides` | drift | `coga/workflows/dream/*.md` | **This run's own `b19e5ba` workaround.** The package deleted these on purpose. They make the dead child-task path look live. Delete. | none |
| C5 `orphan-dream-child-task` | stale | `coga/tasks/dream/validate-drift.md` | Same as K5. This run's `active` child task. | none |
| C6 `claude-md-context-composition-claim` | drift | `CLAUDE.md` + `AGENTS.md` | Same as K8, plus: Common-commands list omits `coga run`, now the entry point for every recipe. Only auto-attach in the codebase is `coga/period-task` (`recurring.py:886-892`). | none |
| C7 `direct-body-skill-copy-divergence` | drift | `coga/skills/direct/body/SKILL.md` | Repo copy has the older escalation rule; packaged distinguishes attended vs queue runs. Load-bearing — every Dream/REM period task runs under `direct/body`. | none |
| C8 `resolve-conflicts-recurring-not-installed` | gap | `coga/recurring/` | Package ships a `resolve-conflicts` recurring template + CLI command; `coga init` never back-fills. Repo has a stale-branch backlog, so the omission is load-bearing. | none |
| C9 `coga-gitignore-orphan-comment-block` | stale | `coga/.gitignore` | Same as K15. Delete the duplicated comment block after the end marker. | none |

**Operator-side note (no PR can fix it):** `coga/.coga/.venv/` vendors a
non-editable coga 0.3.0 whose CLI lacks `run`, `resolve-conflicts`, and
`secret`, while on-PATH `coga` resolves to the editable checkout
(`coga/.coga/COGA_PIN` → `/home/n/Code/claude/coga`). Both report `0.3.0`, so
`--version` hides the skew. If anything resolves `coga` to the vendored binary,
every `coga run <recipe>` in the resynced templates fails. Gitignored — remedy
is an operator `coga init` re-vendor. Surfaced as `human-needed`.

### Merged disposition plan (Phases 2 + 3 deduped)

Proposal PRs (`pr-required`, never auto-merged):
- **PR A** — recurring recipe surface: C1 + C3 + K2 + K11 + K12, and K3's
  repo-side mitigation (move `last_serviced_period` above `### Digest State`).
- **PR B** — Dream template Phases 1/5 (K1/C2) + delete vestigial
  `coga/workflows/dream/` (C4). Based on PR #1's branch, not `main`.
- **PR C** — packaged-copy resyncs: C6/K8 (both `CLAUDE.md` and `AGENTS.md`),
  C7, C9/K15.

Gap draft tickets (`code/with-review`): K3+K4 (upstream digest fixes),
K6+K13 (surprise-lunch reset + worktree prune), K7 (ADK skill disposition),
K10 (Slack enablement), C8 (install resolve-conflicts).
**Do not re-create K9's two existing W30 drafts.**

`human-needed`: K9 (W30 drafts idle a week), K14 (PR #1 unreviewed a full
period), the vendored-venv skew above. K5/C5 is this run's own residue —
resolved in-run rather than deferred.

## Dream Skill: cleanup-orphan-markers

Generated: 2026-07-27T22:54:35+00:00
Task: `recurring/dream`

Result: no-op. No cleanup-eligible processed done tickets still have task directories.

## Dream Run Summary

Generated: 2026-07-27T22:58Z · period 2026-W31 · repo `/home/n/Code/demo-hackathon`

| # | Phase | Result | Detail |
|---|---|---|---|
| 1 | validate-drift | `reported` | 1 issue (0 direct-fix, 0 pr-proposal, 1 human-needed). Dispatch drift hit first — see note below. |
| 2 | knowledge scan | `reported` | 15 findings: 0 extract, 4 drift, 3 stale, 8 gap. |
| 3 | contract audit | `reported` | 9 findings: 6 drift, 2 stale, 1 gap, + 1 operator-side note. |
| 4 | retro/done-ticket | `direct-fixed` | 6 period tickets direct-deleted. 0 knowledge PRs, 0 `## Retro` markers — correct for period tickets. |
| 5 | cleanup-orphan-markers | `no-op` | No cleanup-eligible processed done tickets remain. |
| 6 | disposition | `pr-opened` / `proposed` | 3 proposal PRs, 5 gap draft tickets. |

**Phase 1 dispatch deviation.** The live editable CLI (`/home/n/Code/claude/coga`)
no longer ships `bootstrap/workflows/dream/*`, and both `bootstrap/dream/tasks/*`
skills dropped `script: run.py`. Phases 1 and 5 are now registered recipes.
I restored the workflow files to get Phase 1 running (`b19e5ba`), then reverted
that once the contract audit confirmed the removal was deliberate (`2a47b94`),
and ran both phases via `coga run`. Net repo change from the workaround: none.

### Findings routed

**24 findings across Phases 2–3, deduped to 8 durable artifacts.**

`drift` / `stale` → 3 proposal PRs (all `pr-required`, none auto-merged):

- **[#3](https://github.com/nicktoper/demo-hackathon/pull/3)** — recurring
  recipe surface (C1, C3, K2, K11, K12, K3-mitigation). *Highest value:* the
  five templates lack the `recipe:` key, so `recurring_runner.py:519` falls
  through to `launch_cmd` and spawns a full agent per job — ~1M cache-read
  tokens per sweep for deterministic no-agent work.
- **[#4](https://github.com/nicktoper/demo-hackathon/pull/4)** — Dream Phases
  1/5 recipe model (K1, C2). **Stacked on PR #1**, disjoint hunks of the same
  file; merge #1 first or retarget to `main` if #1 is closed.
- **[#5](https://github.com/nicktoper/demo-hackathon/pull/5)** — context
  composition claim in `CLAUDE.md`/`AGENTS.md` (C6/K8), `direct/body` resync
  (C7), `.gitignore` cleanup (C9/K15).

`gap` → 5 draft tickets (`code/with-review`):

- `fix-coga-digest-state-write-clobbering-and-the-log` (K3+K4)
- `surprise-lunch/reset-the-cli-contract-ticket-to-implement-and-cle` (K6+K13,
  and Phase 1's `stuck-in-progress`)
- `decide-the-fate-of-the-seven-unreferenced-google-a` (K7)
- `enable-a-notification-channel-or-park-the-slack-on` (K10)
- `install-the-resolve-conflicts-recurring-task` (C8)

`extract` → none. Both scans read all six done period tickets in full and found
no durable knowledge; their observations were already captured as K3/K4/K6/K11/K13
and routed above before deletion.

### human-needed / review gates

1. **PR #1 unreviewed for a full period** (K14). It now has PR #4 stacked on it.
   Every weekly sweep re-reports the branch until it is merged or closed.
2. **Two 2026-W30 gap drafts untouched for a week** (K9) —
   `populate-the-base-repo-context-stub`,
   `capture-control-checkout-git-limitation-and-worktr`. Deliberately **not**
   duplicated this run. The week of no decision is itself the signal.
3. **Vendored CLI skew** — `coga/.coga/.venv/` holds a non-editable coga 0.3.0
   whose CLI lacks `run`, `resolve-conflicts`, and `secret`, while on-PATH
   `coga` resolves to the editable checkout. Both report `0.3.0`, so
   `--version` hides it. If anything resolves `coga` to the vendored binary,
   every `coga run <recipe>` in PR #3's resynced templates fails. Gitignored —
   needs an operator `coga init` re-vendor.
4. **PR #2 blocks the ADK skill decision** — settle it before acting on that
   gap ticket; do not open a competing PR.
5. **`surprise-lunch/define-cli-and-configuration-contract` stays
   `in_progress`** (Phase 1's only issue). Dream does not change lifecycle
   state; the reset is tracked in its gap ticket.
