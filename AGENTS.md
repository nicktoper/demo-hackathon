# Agent guide

This repo uses [coga](https://github.com/FastJVM/coga) to coordinate shared
task and context state between humans and agents. Everything coordinated lives
under `coga/`.

## Start here

Run `coga launch bootstrap/orient` to drop into a coga-aware session — the
canonical contexts get composed into the prompt automatically. For ticket-bound
work, prefer `coga launch <slug>` so the ticket's own contexts and current
workflow step are loaded too.

## Common commands

- `coga status` — triage view of all tasks
- `coga ticket "<title>"` — guided task authoring
- `coga create "<title>"` — raw draft create
- `coga dream` — run the Coga cleanup pass now
- `coga mark active <slug>` — activate a draft without launching it (launch activates inline on its own)
- `coga launch <slug>` — start or resume a task (any unique prefix works)
- `coga show <slug>` — read a task's ticket / blackboard / log
- `coga bump <slug>` — advance one workflow step
- `coga mark done <slug>` — finish active or in-progress work
- `coga block --task <slug> --reason "..."` — stop for a concrete answer
- `coga unblock <slug> --answer "..."` — record the answer and resume
- `coga --help` — full CLI surface

## Mental model

The canonical contexts are package-backed and composed automatically; a repo can
override them with local files under `coga/contexts/coga/`. Read in order:

- `principles/SKILL.md` — non-negotiables (markdown-first, fail-loud, classical mode)
- `architecture/SKILL.md` — primitives, planes, prompt composition, locking
- `cli/SKILL.md` — full command reference

These are the exact context refs composed into every launched ticket; if they
disagree with anything else in the repo, they win.

## Don't

- Don't hand-edit `status` / `step` / `workflow` in ticket frontmatter — the
  CLI manages them. Use `coga bump` / `coga block` instead.
- Don't write to the repo-global `coga/log.md` — also CLI-managed.
  Working notes go in the blackboard region of `ticket.md`.
- Don't commit secrets. Use `coga.local.toml` (gitignored) for machine-local
  values, and `env:VAR_NAME` references in `coga.toml` for shared ones.
