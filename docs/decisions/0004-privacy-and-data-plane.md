# ADR 0004 — Local-first, customer-owned blob storage, opt-in content centralization

Date: 2026-05-12
Status: Accepted

## Context

The daemon captures session-level data that necessarily includes prompts (which routinely contain credentials, customer data, internal hostnames, unreleased product details) and file diffs (which are literally proprietary source code). What the daemon sends, where it stores, and who can read it determines whether enterprises will buy at all — and what compliance burden Revelio carries.

Options considered:

- **A. Full content centralization** in Revelio cloud. Maximum analytical leverage. Maximum security-review friction.
- **B. Metadata only** (no prompts, no diffs — just counts, cost, decisions). Cleanest privacy posture. Kills the session-quality moat.
- **C. Tiered with on-device pre-processing.** Default: metadata-only is centralized; content stays local; daemon runs lightweight on-device analysis (redaction, optional quality scoring against rubrics). Customers opt in per-org to ship content with customer-managed keys.
- **D. Customer-owned data plane.** Daemon writes to a customer-owned cloud bucket; Revelio's hosted service reads/queries via federation. Heaviest engineering, cleanest enterprise story.

## Decision

**C as the default privacy stance; D as the architecture for the hosted product:**

- Daemon is local-first. Local SQLite is the canonical store. Code and prompts stay on-device by default.
- Daemon performs on-device PII / secret redaction before any transmission. Hash-logged for customer-side audit.
- Daemon emits metadata-only events to the **customer-owned blob storage** (S3 / GCS) — Revelio's hosted product reads from there; raw content does not enter Revelio's perimeter unless explicitly opted in.
- Content centralization (full prompts, responses, diffs) is per-org opt-in, encrypted at rest with customer-managed keys.
- On-device quality scoring (LLM-as-judge against rubrics) is optional; the **score is the metadata that ships**, not the content.

## Consequences

- Marketing leans into privacy posture: **"We see how, not what."**
- Adapter and daemon engineering must include redaction primitives and right-to-delete at prompt-level granularity (not just user-level).
- Hosted product is a thin layer over the customer's own bucket — billing, dashboards, aggregations.
- On-device LLM-as-judge has a local-CPU / memory footprint. Make it opt-in or async / batched to avoid the product-killing "this slowed my laptop down" failure mode.
- Becoming a SOC2 / ISO / HIPAA-adjacent vendor is deferred (or avoided entirely) by virtue of not centralizing raw content by default.
- Default deployment requires customer to bring their own cloud bucket — this is friction that has to be mitigated by tooling (`revelio init` should walk through bucket provisioning).
