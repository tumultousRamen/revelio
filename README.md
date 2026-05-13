# Revelio

> Reveal what is hidden in your team's AI coding sessions.

Vendor-neutral observability and ROI dashboard for AI coding agents — Claude Code, Codex CLI, Cursor, Windsurf, opencode, and the long tail. Engineering managers see ROI and adoption across every tool their team uses. Engineers get personal coaching feedback. Self-hostable OSS daemon + hosted SaaS for team-scale analytics.

## Status

Idea phase. This repo currently holds design decisions only — no code yet.

## Read first

- [`CONTEXT.md`](./CONTEXT.md) — what this project is and how to navigate it
- [`docs/decisions/`](./docs/decisions/) — architecture decision records (ADRs) for major choices
- [`docs/parking-lot.md`](./docs/parking-lot.md) — open strategic questions

## Architecture sketch

```
[ Dev's machine ]                  [ Customer cloud ]            [ Revelio cloud ]
  AI coding tool   →   local   →   per-customer        →   (metadata only by default;
  (Claude Code         daemon       blob storage              content opt-in)
   etc.)               (redact +    (customer owns)
                       normalize)
```

- Per-tool adapters → canonical event schema (OTel GenAI conventions + Revelio envelope)
- Local-first: code and prompts stay on-device by default
- Replicates to a per-customer-scoped cloud bucket they own
- On-device PII redaction + optional on-device quality scoring
