# Puzzle External API — Versioning Policy

> **Status:** Draft for review · **Applies to:** the REST API at `/rest/*` and its generated OpenAPI spec

## Version identity

The API has exactly one version dimension: the **major version in the URL** (`https://api.puzzle.io/rest/v0`). One OpenAPI document exists per major version, served at `/rest/<version>/openapi.json`; the `paths` within it are unversioned. The document's `info.version` records the spec revision within a major and moves freely; the URL version moves rarely and deliberately. **Individual endpoints are never versioned** — there is no `/v1/bills` next to `/v0/bills`, ever.

## Change taxonomy

Every API change is exactly one of:

- **Editorial** — descriptions, examples, titles. No behavioral meaning. Ship freely.
- **Additive** — new endpoints; new *optional* request fields; new response fields; new enum values. Evolves the current major in place. Ship with a changelog entry.
- **Breaking** — removing or renaming endpoints, fields, or enum values; type/format changes; optional→required; narrowing accepted inputs; semantic changes to existing fields or status codes; auth, error-shape, or pagination changes. **Never ships inside a stable major.**

Clients are expected to be tolerant: unknown response fields and unknown enum values must not be treated as errors. We state this in the public docs; it is what makes "additive" safe.

## Rules for a breaking need

When a change looks breaking, walk this ladder — stopping at the first rung that works:

1. **Dissolve it additively.** Add the new field beside the old; introduce a new resource noun instead of mutating an existing one.
2. **Deprecate, don't remove.** Mark the old element `deprecated: true` in the contract (renders as a badge in the docs), announce a sunset date in the changelog, keep serving it. Deprecation queues removal; it never performs it.
3. **Add it to the next-major wishlist.** Actual removals and reshapes ship only in a coordinated whole-surface major bump: `/rest/v1` cut for *every* endpoint at once, v0 dual-served through a published sunset window (minimum 12 months for a stable major).

## Current status: v0

`v0` is **pre-stable**: breaking changes are permitted with notice (changelog + direct partner communication, minimum 30 days), and no major bump is owed for them. This window is for fixing what the old spec-generation era got wrong — it will not be extended by habit.

**v1 graduation criteria** (all required): production drift detection quiet over a full close cycle; `securitySchemes` and error shapes modeled in the spec; this policy published in the developer docs; sign-off that we would honor the stable-major rules for every endpoint as-is.

## Enforcement

The spec is generated deterministically from the Zod contracts, so classification is mechanical, not honor-system: CI diffs the generated OpenAPI document on every PR and labels the change editorial / additive / breaking. A PR carrying a breaking diff fails unless it carries the matching ritual (deprecation marker, next-major label, or — during v0 only — an approved notice plan). Production drift detection remains the backstop for what review misses.

## Communication

Every additive or breaking change lands in the public changelog, keyed to `info.version`. Deprecations state their sunset date at announcement time. A new major ships with a migration guide enumerating every breaking diff — generated from the spec diff, not written from memory.
