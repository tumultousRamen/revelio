# Revelio — project context

Vendor-neutral observability and ROI dashboard for AI coding agents. Read [`README.md`](./README.md) first.

## Who it serves

- **Primary buyer (phase 1)**: VPs Eng / Engineering Managers justifying AI-tooling spend.
- **Secondary user (phase 2)**: individual engineers wanting personal coaching feedback on their AI-assisted coding sessions.

## Product layers

- **Wedge** — seat utilization and AI-spend rationalization. Easy data, immediate ROI story, gets you in the door.
- **Promise** — outcome correlation. Tie AI-tool usage to engineering outcomes (cycle time, defect rate, throughput).
- **Moat** — code-level attribution. Per-commit / per-PR attribution: what % was AI-generated, mapped to downstream signals (bugs, reverts, churn).

## Distribution

OSS-core + hosted SaaS:

- **OSS**: local daemon, all tool adapters, on-device redaction, single-user dashboard. "Be your own engineering manager."
- **Hosted**: multi-user aggregation, manager dashboards, cohort and benchmark analytics, hosted blob replication, SSO/SCIM. Per-seat pricing.

## Data flow

1. Local daemon on dev machine subscribes to each AI tool's session data (JSONL transcripts + hooks where available).
2. Daemon redacts on-device (PII, secrets) and normalizes into Revelio's canonical event schema.
3. Local SQLite is the canonical store.
4. Replicates to a per-customer-owned cloud bucket (S3/GCS) — customer-controlled data plane.
5. Hosted product queries the customer's bucket; raw content does not enter Revelio's perimeter unless explicitly opted in.

## Tool-data asymmetry (load-bearing)

Not every tool exposes the same depth of session data. The product accepts and communicates this rather than hiding it.

- **Deep tier** (agentic CLI): Claude Code, Codex CLI, opencode, Aider, Cline. Full session-level data — prompts, responses, tool calls, file edits, accept/reject decisions. Powers the session-quality and coaching layer.
- **Aggregates tier** (IDE assistants): Cursor, Windsurf, Copilot. Admin-API-only ingestion — utilization, acceptance counts, model mix, cost. Powers the utilization-and-spend wedge.

## Tech stack (v0)

- Daemon: TypeScript on Bun runtime. Single-file binary via `bun build --compile` for distribution.
- Dashboard: Next.js (TypeScript).
- Canonical schema: OpenTelemetry GenAI semantic conventions + Revelio-namespaced envelope.
- Local store: SQLite.
- Customer-side cloud store: S3 / GCS (customer-owned).

## Decision log

ADRs live in [`docs/decisions/`](./docs/decisions/):

- [0001 — Buyer and wedge](./docs/decisions/0001-buyer-and-wedge.md)
- [0002 — Data acquisition and richness asymmetry](./docs/decisions/0002-data-acquisition-and-asymmetry.md)
- [0003 — Canonical schema](./docs/decisions/0003-canonical-schema.md)
- [0004 — Privacy and data plane](./docs/decisions/0004-privacy-and-data-plane.md)
- [0005 — Name and positioning](./docs/decisions/0005-name-and-positioning.md)

Open strategic questions: [`docs/parking-lot.md`](./docs/parking-lot.md).

## Issue tracking

Per workspace convention, issues live under `.scratch/<feature-slug>/` as markdown files.
