# Daily digest spool

Producer/consumer queue for `coga digest`. Producers append one JSONL record
at the **bottom** of `## Spool (pending)`; the single consumer (`coga digest`)
advances the `consumed_through:` watermark to the newest record and trims the
consumed prefix, always keeping the newest record in place as an *anchor*.

This file is marked `merge=union` (`.gitattributes`) so two clones appending
concurrently merge without conflict. Together with the top-trim/bottom-append
shape (deletes and appends sit in disjoint hunks separated by the anchor), that
makes the spool mergeable by construction with no lock — see the `coga/sync`
context. The git high-water mark lives separately in the digest ticket's
`### Digest State`, not here.

## Spool (pending)


consumed_through: 5f013a22077d
{"id":"5f013a22077d","ts":"2026-07-20T16:45","project":"demo-hackathon","kind":"done","detail":"claude finished: execute → done ✅ — 1 live branch: feat/surprise-lunch-cli rebased-local (clean, 98 tests pass), not pushed (no PR remote). Unblocks surprise-lunch/define-cli-and-configuration-contract open-pr.","ticket":"recurring/rebase-stale-worktrees","owner":"nicktoper"}
{"id":"501ef0ccdf99","ts":"2026-07-20T16:45","project":"demo-hackathon","kind":"done","detail":"→ done (script)","ticket":"recurring/digest","owner":"nick"}
{"id":"7ca78ef90f03","ts":"2026-07-20T16:45","project":"demo-hackathon","kind":"done","detail":"→ done (script)","ticket":"recurring/skill-update","owner":"nicktoper"}
{"id":"f14746303110","ts":"2026-07-20T16:45","project":"demo-hackathon","kind":"done","detail":"→ done (script)","ticket":"recurring/blocker-reminders","owner":"nicktoper"}
{"id":"c089716062f3","ts":"2026-07-20T16:59","project":"demo-hackathon","kind":"done","detail":"claude finished: execute → done ✅ — Dream 2026-W30 complete: 6 period tickets retired, 1 drift PR (#1), 2 gap drafts, validate clean.","ticket":"recurring/dream","owner":"nicktoper"}
