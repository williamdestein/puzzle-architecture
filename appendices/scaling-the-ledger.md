# Appendix: scaling-the-ledger

> Staged 2026-07-24. Not yet merged into architecture.md.
> Primary source: `ledger/docs/decisions/001-go-ledger.md` (ADR, 2025-07-16,
> Accepted).

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
fixes it. The Go effort attacks exactly that constraint: language-level
concurrency (which the GIL forbade) plus the decomposition — `gopzevents` as
a *stateless* event-to-journal transformer (horizontally scalable by
construction) and `gopzdata` for lock-free reference reads — to shrink what
actually has to serialize inside the company lock.

**Verdict:** not "vertical because horizontal defeated them," but "the hot
constraint is a per-company ordering invariant that horizontal scaling
cannot touch, and Python blocked the only lever that can." The honest caveat
the ADR itself flags as Neutral: the Go ledger uses *the same Postgres
database* as the Python ledger — the one genuinely vertical, genuinely
shared chokepoint survives the rewrite intact.
