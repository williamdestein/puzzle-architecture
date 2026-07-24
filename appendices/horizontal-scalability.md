# Appendix: horizontal-scalability

> Staged 2026-07-24. Not yet merged into architecture.md.
> Companion to [scaling-the-ledger]. Sources: org-wide code search
> (2026-07-24); `ledger/docs/decisions/001-go-ledger.md`.

**Q: Platforms like QuickBooks and Yahoo Mail scale horizontally with
"farms"/"clusters" — a load balancer, app servers, and a database per cell,
plus a directory mapping each tenant to its cell. Is Puzzle ready for, or
even thinking about, that sort of horizontal scalability?**

**On the "is anyone thinking about it" axis, the evidence says no.**
Searching the entire org: "shard" appears only as CI test-sharding; Citus,
Spanner, and AlloyDB appear nowhere. There is no directory service, no
cell/farm concept, no tenant-to-cluster routing in any repo or doc. The one
recent moment where the question could have been raised — the go-ledger ADR —
explicitly kept "the same Postgres database" and filed that under *Neutral*:
data-tier scale-out was not weighed as a consequence. Meanwhile the symptoms
of a single hot database are already in the commit log: replica-lag
workarounds, two cache tiers, request batching, per-endpoint throttles
guarding the ledger.

**Three counterweights to the alarm:**

1. **The scale math differs from the precedents.** QuickBooks and Yahoo Mail
   farmed because tens of millions of consumer tenants hit write-heavy
   stores in an era when one database box had a hard ceiling. Puzzle is B2B
   accounting — tenants are companies, not consumers — and a well-partitioned
   single Postgres with replicas and aggressive caching carries B2B SaaS far
   into six figures of tenants. The current design is the correct pre-farm
   stage, with runway left.

2. **When the ceiling comes, farms are no longer the only door.** The 2005
   answer to "one DB can't hold everyone" was cells-with-a-directory because
   nothing else existed. The 2026 menu includes declarative partitioning,
   Citus-style distributed Postgres, AlloyDB, and Spanner — scale-out inside
   the data tier, with no per-cell fleets and no directory service to
   operate. On GCP, that is the likelier path.

3. **Cell-readiness is a discipline, not a deployment — and Puzzle mostly
   has it.** Everything is company-scoped; there are no cross-company
   transactions; event batches, locks, and caches all key on company. A
   tenant whose data forms a clean partition can be moved to a cell later.
   By that test the platform is closer to farm-able than most systems its
   age, with exactly two entanglements that would block it: **accounting
   reading the gateway's DB replica directly** (in a cell world, "which
   replica?" has no answer without a directory — that tech debt is also cell
   debt), and **firm-level rollups** (a Firm's companies would need
   co-location or scatter-gather).

**Verdict:** right that nobody is designing the directory, and right that
the shared ledger Postgres is the structural ceiling — but the per-company
discipline means the expensive precondition of cell architecture is already
paid. What's missing is a written scaling thesis: *when* the single-DB
runway ends, and *which* door to take. That is a one-page ADR someone should
write before the pager writes it for them.
