# ADR 0003 — Hybrid OTel GenAI + Revelio envelope as the canonical event schema

Date: 2026-05-12
Status: Accepted

## Context

Per-tool adapters (see [ADR 0002](./0002-data-acquisition-and-asymmetry.md)) must normalize into a single canonical event model. The canonical schema is the contract between every adapter, every analytics query, every dashboard chart, every future tool added. It is harder to change than the database and locks in years of downstream design.

Options considered:

- **A. Roll our own proprietary schema.** Maximum control, zero interop, all adapter cost on us.
- **B. OpenTelemetry GenAI semantic conventions as canonical.** Standard, future-proof, free interop with collectors and storage. But: GenAI conventions are partly experimental and do not cover agent-level concepts well (no standard for edit accept/reject, permission decisions, session compaction, multi-agent sub-sessions).
- **C. Hybrid.** OTel-compatible attributes where they exist (`gen_ai.system`, `gen_ai.request.model`, `gen_ai.response.usage.*`); Revelio-namespaced envelope (`revelio.session.*`, `revelio.decision.*`, `revelio.edit.*`) for what OTel doesn't yet cover. OTLP as wire format.
- **D. Defer.** Capture raw, schematize later. The standard "we'll fix it in v2" trap — leads to a permanent `raw_events` JSON blob nightmare and a migration that never happens.

## Decision

**Hybrid (C), with deferred sequencing of schema population:**

- Commit to the hybrid architecture **now**.
- **Populate** the schema by mapping Claude Code first as the reference adapter — let the schema crystallize against the richest real case rather than designing it in the abstract.
- Codex CLI second; the delta exposes tool-specific cruft vs. genuinely canonical concepts.
- By the third adapter (opencode or Cursor's admin API), the schema should be stable enough to publish as a versioned package.
- Pin to a specific OTel GenAI conventions release version; document the upgrade path; assume attribute renames within 18 months.

## Consequences

- We get OTel ecosystem benefits (collectors, processors, exporters) for free where conventions overlap.
- We can evolve agent-level concepts (`revelio.*`) on our own timeline without waiting for upstream SIG.
- The Claude Code adapter is a near-passthrough; cost of adapter work is concentrated on tools without native OTel.
- Schema migration risk lives at the `gen_ai.*` boundary — track upstream releases.
- The canonical schema package likely lives in OSS (interop and ecosystem value); to be confirmed in [parking-lot.md §1](../parking-lot.md).
