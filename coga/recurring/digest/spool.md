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

consumed_through:
{"id":"d7b0da86b7b6","ts":"2026-07-20T16:41","project":"demo-hackathon","kind":"done","detail":"→ done (script)","ticket":"recurring/branch-sweep","owner":"nicktoper"}
{"id":"7c06f73aafbd","ts":"2026-07-20T16:41","project":"demo-hackathon","kind":"done","detail":"→ done (script)","ticket":"recurring/autoclose-merged","owner":"nicktoper"}
{"id":"5f013a22077d","ts":"2026-07-20T16:45","project":"demo-hackathon","kind":"done","detail":"claude finished: execute → done ✅ — 1 live branch: feat/surprise-lunch-cli rebased-local (clean, 98 tests pass), not pushed (no PR remote). Unblocks surprise-lunch/define-cli-and-configuration-contract open-pr.","ticket":"recurring/rebase-stale-worktrees","owner":"nicktoper"}
