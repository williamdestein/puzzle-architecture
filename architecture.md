# Puzzle Platform — Architecture

> **Status:** Draft · **Owner:** Bill DeStein · **Last updated:** 2026-07-24
>
> Sections merged so far: Services and Repositories.

---

## Table of Contents

<!-- Regenerated whenever a chapter is added or an appendix is merged. Most
     renderers (GitHub, VS Code, Pandoc with --toc) also auto-derive a TOC
     from the headings; this manual list doubles as the document outline. -->

1. [Overview](#overview)
2. [Services and Repositories](#services-and-repositories)
3. [Architecture](#architecture)
   1. [System Context](#system-context)
   2. [Component View](#component-view)
4. [Data Stores](#data-stores)
5. [Events and Messaging](#events-and-messaging)
6. [Key Flows](#key-flows)
7. [External API](#external-api)
8. [Deployment](#deployment)
9. [Cross-Cutting Concerns](#cross-cutting-concerns)
10. [Open Questions](#open-questions)
11. [References](#references)

---

## Overview

<!-- to be written -->

## Services and Repositories

The accounting engine spans six repositories:

| Repo | Language | Role in the engine |
|------|----------|--------------------|
| `gateway` | TypeScript | Ingests integration data, builds the financial graph, emits ledger events, serves the APIs |
| `ledger` | Python | The general ledger — append-only, bitemporal source of truth; owns the journal and chart of accounts |
| `goledger` | Go | Ledger performance sidecar — processes events for custom-CoA companies, payroll journal mappings, syncs to the data service |
| `accounting` | Python | Accrual computations — revenue schedules from invoice/subscription data |
| `gopzevents` | Go | Events service — the eventing backbone for the Go-era stack |
| `gopzdata` | Go | Data service — company data store that goledger syncs into |

Three of the six own PostgreSQL schemas (`gateway`, `ledger`, `accounting`);
`goledger` rides on the ledger's stores. Data flows from the gateway to the
ledger, goledger, and accounting services over REST, plus an event-sourcing
Pub/Sub path (see [Data Stores](#data-stores) and
[Events and Messaging](#events-and-messaging)).

A boundary note: the first four repositories are unambiguously the engine.
`gopzevents` and `gopzdata` are infrastructure *for* the engine — born of the
custom-CoA / goledger effort — so whether they count as "engine" or "engine
plumbing" is a judgment call; they are included here because the ledger's own
documentation treats the trio (goledger, events, data) as one architectural
move. Counting compiled-in code rather than deployed services, the engine
also carries `jigsaw` (shared Python), `gopuzzle` (shared Go), `sqcache`
(cache layer), and the `temporal_tables_puzzle` fork that gives the ledger
its bitemporality — libraries rather than services, but engine code all the
same.

## Architecture

### System Context

<!-- to be written -->

### Component View

<!-- to be written -->

## Data Stores

<!-- to be written -->

## Events and Messaging

<!-- to be written -->

## Key Flows

<!-- to be written -->

## External API

<!-- to be written -->

## Deployment

<!-- to be written -->

## Cross-Cutting Concerns

<!-- to be written -->

## Open Questions

<!-- to be written -->

## References

<!-- to be written -->
