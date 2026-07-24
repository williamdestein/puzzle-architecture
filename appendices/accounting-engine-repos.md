# Appendix: accounting-engine-repos

> Staged Q&A, 2026-07-24. Not yet merged into architecture.md.

**Q: What are the repos that form the accounting engine?**

Six repos:

| Repo | Language | Role in the engine |
|------|----------|--------------------|
| `gateway` | TypeScript | Ingests integration data, builds the financial graph, emits ledger events, serves the APIs |
| `ledger` | Python | The general ledger — append-only, bitemporal source of truth; owns the journal and chart of accounts |
| `goledger` | Go | Ledger performance sidecar — processes events for custom-CoA companies, payroll journal mappings, syncs to the data service |
| `accounting` | Python | Accrual computations — revenue schedules from invoice/subscription data |
| `gopzevents` | Go | Events service — the eventing backbone for the Go-era stack |
| `gopzdata` | Go | Data service — company data store that goledger syncs into |

That's the set whose databases and call graph we mapped: three of them own
Postgres schemas (gateway, ledger, accounting), goledger rides on the ledger's
stores, and the data flows gateway → ledger/goledger/accounting via REST plus
the event-sourcing Pub/Sub path.

A boundary note: the first four are unambiguously the engine. `gopzevents` and
`gopzdata` are infrastructure *for* the engine — born of the custom-CoA /
goledger effort — so whether they're "engine" or "engine plumbing" is a
judgment call; include them, since the ledger's own docs treat the trio
(goledger, events, data) as one architectural move. Counting compiled-in code
rather than deployed services, the engine also carries `jigsaw` (shared
Python), `gopuzzle` (shared Go), `sqcache` (cache layer), and the
`temporal_tables_puzzle` fork that gives the ledger its bitemporality —
libraries, not services, but engine code all the same.
