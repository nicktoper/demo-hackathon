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



consumed_through: 974b8ca7a0da
{"id":"974b8ca7a0da","ts":"2026-07-27T15:33","project":"demo-hackathon","kind":"done","detail":"claude finished: execute → done ✅ — Rebase sweep 2026-W31: nothing rebased. Only live branch feat/surprise-lunch-cli has 0 own commits — the surprise-lunch implementation is not in this repo (lost with a deleted /tmp clone). Ticket define-cli-and-configuration-contract needs to go back to implement, not open-pr.","ticket":"recurring/rebase-stale-worktrees","owner":"nicktoper"}
{"id":"48182376364c","ts":"2026-07-27T15:35","project":"demo-hackathon","kind":"done","detail":"claude finished: flush → done ✅","ticket":"recurring/digest","owner":"nick"}
{"id":"1ab8a211ac0d","ts":"2026-07-27T15:37","project":"demo-hackathon","kind":"done","detail":"claude finished: update → done ✅","ticket":"recurring/skill-update","owner":"nicktoper"}
