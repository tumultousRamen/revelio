# Parking lot — open strategic questions

Captured here so we don't forget. None are blocking for v0 engineering but all will need resolution before launch.

## 1. OSS / SaaS boundary

What lives in the open-source release vs. cloud-only? Classic open-core trap: too generous → no one pays; too stingy → no one adopts.

Working hypothesis:

- **OSS**: local daemon, all tool adapters, on-device redaction, single-user dashboard, canonical schema package. "Be your own engineering manager."
- **Cloud-only**: multi-user aggregation, manager dashboards, cohort and benchmark analytics, hosted blob replication, billing-grade ROI reports, SSO / SCIM.

Open: does the canonical schema package live in OSS or only in the cloud product? (Strong lean: OSS — interop and ecosystem value.)

## 2. Pricing model

Per-seat? Per ingested event? Percentage of AI-tooling spend?

- **Per-seat** — boring-correct B2B SaaS pattern; easy to procure; predictable revenue.
- **% of AI spend** — more compelling CFO story ("we save you 10% of your AI budget for X% of it"), but harder to model and to invoice.
- **Per-event** — aligns with usage but creates incentive friction (customers throttle ingest, which kneecaps the product).

Lean: per-seat for simplicity, with usage-based tiers above a threshold. Revisit after first 5 paying customers reveal actual ingest volumes.

## 3. The productivity paradox honesty problem

What does the product *say* when the data shows a team's AI investment isn't moving the needle?

The defensible answer is **"we tell the truth."** But if our business model assumes AI tooling is universally valuable, we've built a tool that has to flatter every customer — and is therefore worthless. We need a stance.

Needs an explicit founder-level position and probably a manifesto-style external doc by the time we have a public site.

## 4. PII / secret redaction stance

Strict-by-default (allowlist) or permissive-by-default (denylist)?

Lean: **strict**. The cost of one leaked customer secret in a Revelio-hosted prompt is existential.

Open: classifier accuracy threshold? What's the false-positive escape valve when the redactor over-eats a valid prompt? Where does the redactor itself run — pure regex, small on-device classifier, or hybrid?

## 5. IC-facing product surface and timing

Phase 2, but UI principles should be set now to avoid painting into a corner.

Open: does the IC dashboard live inside the OSS distribution, the cloud product, or both? Lean: OSS (single-user) covers most of it; the hosted product offers cross-engineer comparison and coaching (with privacy controls).

## 6. Hooks-vs-JSONL deduplication

Both capture overlapping data (tool calls, edits). The daemon needs explicit de-dup logic to avoid double-counting and to choose canonical source per event type.

Working hypothesis: hooks are authoritative for **lifecycle and decisions** (session start/end, accept/reject, permission outcomes); JSONL is authoritative for **content** (prompts, responses, tool I/O). When both exist, hook event references the JSONL line by UUID; daemon merges at normalization time.

## 7. Schema versioning and migration

The canonical schema *will* change. We need a migration story before the first paying customer, not after.

- Per-event `schema_version` attribute on every record.
- Adapter version recorded alongside.
- Migrations as forward-only transforms in the daemon at write-to-cloud time (cheap) rather than at read-from-cloud time (expensive at query scale).

## 8. Cross-platform daemon footprint

The daemon runs on every developer's machine — macOS, Linux, Windows (Windows is real for enterprise). Bun's cross-compile story is good on macOS / Linux, less mature on Windows. May need a Go port earlier than expected if Windows adoption matters for the first design partners.
