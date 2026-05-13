# ADR 0002 — Per-tool adapters with rich-data asymmetry

Date: 2026-05-12
Status: Accepted

## Context

The product must ingest session data from many AI coding tools — Claude Code, Codex CLI, Cursor, Windsurf, opencode, Aider, Cline, and the long tail. Each tool has a wildly different data surface (verified against training-data, not live docs — re-verify before parser code):

- **Claude Code**: rich. Local JSONL at `~/.claude/projects/<slug>/<uuid>.jsonl`, nine hook events with typed stdin JSON, native OpenTelemetry emission (`CLAUDE_CODE_ENABLE_TELEMETRY=1`), Admin API for org-level usage (`usage_report/claude_code`).
- **Codex CLI**: medium. Local rollout JSONL at `~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl`, MCP server config as hook surface, no native OTel, no CLI-specific admin API.
- **Cursor**: thin. Admin API returns **aggregates only** (no prompt content). Local `state.vscdb` SQLite is undocumented and unstable. No first-class hook surface for AI events. No OTel.
- **Windsurf**: thinnest. Admin dashboard but no documented REST export. Post-Cognition-acquisition surface unknown.
- **opencode**: open source. Plugin SDK with hooks; TUI/server split with a server API.

Alternative approaches considered and rejected:

- **Git/PR-side only** (analyze commits/PRs without tool-side instrumentation): too lossy — kills the session-quality moat.
- **Vendor-API-only aggregator**: fragile; vendors can revoke access and ship competing dashboards.
- **Per-tool IDE extensions we ship ourselves**: massive engineering cost; some forks block extensions; doesn't scale to the long tail.
- **Lowest-common-denominator schema** (only metrics every tool supports): throws away ~80% of the value of agentic CLI tools to maintain false uniformity.

## Decision

**Per-tool adapters running inside a local daemon. Each adapter captures both JSONL transcripts and hooks/events where the tool exposes them. Two-tier feature set on top:**

- **Deep tier** (agentic CLI cohort): Claude Code, Codex CLI, opencode, Aider, Cline. Full session-level data — prompts, responses, tool calls, file edits, accept/reject decisions, cost. Powers the session-quality and coaching layer.
- **Aggregates tier** (IDE-based assistants): Cursor, Windsurf, Copilot. Admin-API-only ingestion — utilization, acceptance counts, model mix, cost. Powers the utilization-and-spend wedge.

Product marketing explicitly communicates the asymmetry — we go deep where the data allows, and we are honest about what the rest of the tools will let us measure.

**First reference adapter: Claude Code** (richest surface; native OTel; designing the canonical schema against this case forces us through the hardest mapping).

## Consequences

- Per-tool engineering cost is permanent. Hire and build for adapter velocity.
- Customer marketing must clearly call out which tools are "deep tier" vs "aggregates tier" — silent gaps would kill credibility on comparison.
- Cursor and Windsurf customers see less rich dashboards. This is a fact about the world, not a product gap; we should not paper over it.
- The schema (see [ADR 0003](./0003-canonical-schema.md)) must accommodate both tiers without making the rich tier feel impoverished.
- Dual-capture (JSONL + hooks) per tool means adapters carry meaningful complexity. Worth it: hooks capture lifecycle and accept/reject decisions that aren't recoverable from JSONL alone.
