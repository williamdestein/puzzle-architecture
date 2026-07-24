# Appendix: scaling-the-ledger

> Staged 2026-07-24. Not yet merged into architecture.md.
> Primary sources: `ledger/docs/decisions/001-go-ledger.md` (ADR, 2025-07-16,
> Accepted); `goledger/locks/README.md`.

**Q: Is the ledger CPU-bound, and is the Go rewrite a form of vertical
scaling that suggests the ledger team has not solved horizontal
scalability?**

Half right — and the ledger team wrote down the answer in an ADR. Scoring the
three claims against it:

**"The ledger is CPU-bound" — partially confirmed.** The ADR names Python's
GIL explicitly: "Python's architectural choices, including the Global
Interpreter Lock, prevent true concurrency at the language level," and
expects Go to be "significantly faster." Per-process compute and concurrency
limits are a stated motivation — but the *secondary* one. The primary driver
is that custom-CoA (BYOCOA) support "requires a significant rearchitecture":
the Python ledger's event processing bakes in assumptions about which
accounts exist, "affecting almost every event." The rewrite was forced by a
functional requirement; the speed was opportunistic ("we wanted to take the
opportunity").

**"The rewrite is vertical scaling" — fair as far as it goes.** Raising
per-worker throughput and unlocking multicore concurrency inside a process
is spending the vertical budget better.

**"They haven't solved horizontal scalability" — the framing misses; the
ADR's key sentence says why:** *"the current Python Ledger always locks an
entire company when processing events, and processes them sequentially. We
want to enable the concurrent processing of events for a single company."*

Scaling *across* companies is the easy, presumably-solved dimension —
companies are independent books, so work partitions naturally and horizontal
scaling works. The unsolved problem is *within* one company, and it is not a
horizontal-scaling problem at all: a double-entry journal demands ordered,
consistent application of events, so the safe-and-simple answer is a
whole-company lock — and adding workers does nothing when they all queue on
the same lock. The biggest company is the slowest company, and no fleet size
fixes it.

**What "concurrent processing of events for a single company" actually
means: concurrency between events, not within an event.** No single
request or event is decomposed into parallel parts — one event's
transformation into a journal entry is small, sequential work. What changes
is the collision radius. The Python ledger takes one lock on the entire
company and processes its events strictly one at a time, end to end.
goledger replaces that with fine-grained, Redis-backed distributed locks —
its `locks` package hosts `EventsLock`, `BatchLock`, `JobLock`, and
`BalanceLock` as separate scopes — so two batches for the same company that
don't touch the same contended resource no longer wait on each other; only
operations that genuinely conflict (e.g., updating the same balance)
serialize, under the narrowest lock that protects the invariant. Two things
make that concurrency real: Go's runtime (under Python's GIL, one process
could not execute two events' CPU work simultaneously even with finer
locks), and the decomposition — `gopzevents` as a *stateless*
event-to-journal transformer is the naturally parallel middle of the
pipeline, with `gopzdata` providing lock-free reference reads; the
order-sensitive tail (journal persistence, balance updates) is where the
narrow locks still impose sequence.

**Verdict:** not "vertical because horizontal defeated them," but "the hot
constraint is a per-company ordering invariant that horizontal scaling
cannot touch, and Python blocked the only lever that can." The honest caveat
the ADR itself flags as Neutral: the Go ledger uses *the same Postgres
database* as the Python ledger — the one genuinely vertical, genuinely
shared chokepoint survives the rewrite intact.
