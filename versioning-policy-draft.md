# Puzzle External API — Versioning Policy

> **Status:** Draft for review · **Applies to:** the REST API at `/rest/*` and its generated OpenAPI spec

## API versions and endpoint versions

Two notions of "version" come up when discussing an API, and this policy deliberately supports only one of them.

An **API version** is a version of the entire surface — every endpoint at once. It lives in the base URL (`https://api.puzzle.io/rest/v0`) and, in the OpenAPI document, in the `servers` entry; the `paths` within the document are unversioned. One OpenAPI document exists per API version, served at `/rest/<version>/openapi.json`. The URL version moves rarely and deliberately.

Within an API version, the document's `info.version` records spec revisions as `<major>.<minor>.<patch>`: the major matches the URL version, the minor moves on additive changes, and the patch moves on editorial ones. The version is a function of the spec's *content*, never of when it was generated — the document is regenerated constantly (every deploy, every CI build), and two byte-identical documents always carry the same version. Because bumps track the change taxonomy, the version means something: a partner comparing two minors knows capabilities were added in between, not merely that time passed.

An **endpoint version** is the other pattern, and you have likely used APIs built on it: the reference docs show `/v1/bill` and `/v2/bill` as live routes side by side, each endpoint carrying its own version number and moving on its own schedule. **We do not do that.** Endpoints here are never individually versioned — an endpoint's "version" is simply the API version it lives in. What those platforms express as "v2 of an endpoint" is expressed in this policy as the endpoint's evolution *within* an API version: the change is made additive instead (new optional fields, or a new sibling endpoint under its own noun), or the old element is deprecated with a sunset date, or the change waits for the next whole-surface API version (the ladder in "Rules for a breaking need"). An endpoint's history is visible as its field set and its deprecation badges — never as a number in its path.

## Change taxonomy

Every API change is exactly one of:

- **Editorial** — descriptions, examples, titles. No behavioral meaning. Ship freely.
- **Additive** — new endpoints; new *optional* request fields; new response fields; new enum values. Evolves the current major in place. Ship with a changelog entry.
- **Breaking** — removing or renaming endpoints, fields, or enum values; type/format changes; optional→required; narrowing accepted inputs; semantic changes to existing fields or status codes; auth, error-shape, or pagination changes. **Never ships inside a stable major.**

Clients are expected to be tolerant: unknown response fields and unknown enum values must not be treated as errors. We state this in the public docs; it is what makes "additive" safe.

Additive changes are contract-first: the field or endpoint is added to the Zod contract (and thus the published spec) before or with the code that puts it on the wire. A field observed on the wire that the contract does not model is not an additive change — it is a process failure, and drift detection will flag it as one.

## Rules for a breaking need

When a change looks breaking, walk this ladder — stopping at the first rung that works:

1. **Make it additive instead.** Add the new field beside the old; introduce a new resource noun instead of mutating an existing one. Most breaking needs stop being breaking here.
2. **Deprecate, don't remove.** Mark the old element `deprecated: true` in the contract (renders as a badge in the docs), announce a sunset date in the changelog, keep serving it. Deprecation queues removal; it never performs it.
3. **Add it to the next-major wishlist.** Actual removals and reshapes ship only in a coordinated whole-surface major bump: `/rest/v1` cut for *every* endpoint at once, v0 dual-served through a published sunset window (minimum 12 months for a stable major).

## Current status: v0

`v0` is **pre-stable**: breaking changes are permitted with notice (changelog + direct partner communication, minimum 30 days), and no major bump is owed for them. This window is for fixing what the old spec-generation era got wrong — it will not be extended by habit.

**v1 promotion criteria** (all required): production drift detection quiet over a full close cycle; `securitySchemes` and error shapes modeled in the spec; this policy published in the developer docs; sign-off that we would honor the stable-major rules for every endpoint as-is.

## Enforcement

The spec is generated deterministically from the Zod contracts, so classification is mechanical, not honor-system: CI diffs the generated OpenAPI document on every PR and labels the change editorial / additive / breaking. A PR carrying a breaking diff fails unless it carries the matching ritual (deprecation marker, next-major label, or — during v0 only — an approved notice plan). The same gate enforces the `info.version` bump: an additive diff requires a minor bump, an editorial diff a patch bump, and no diff no bump — the version constant lives in the repo and moves in the same PR as the change it describes. Production drift detection remains the backstop for what review misses.

## Communication

Every additive or breaking change lands in the public changelog, keyed to `info.version`. Deprecations state their sunset date at announcement time. A new major ships with a migration guide enumerating every breaking diff — generated from the spec diff, not written from memory.
