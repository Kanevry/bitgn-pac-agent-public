# BitGN PAC Agent — Project Brief

AI agent for the BitGN Personal Agent Challenge (PAC). First place at the Vienna hub
(PAC 2026, AI Factory Vienna, 11 April 2026). Code is published as reference.

See [`README.md`](README.md) for the full architectural overview, defense-layer
documentation, and setup instructions.

## Stack

- **Language**: TypeScript (ESM, strict mode)
- **Runtime**: Node.js 24+ via `tsx`
- **Package Manager**: pnpm
- **LLM Driver**: Vercel AI SDK v6 (`ai`) with `@ai-sdk/anthropic` + `@ai-sdk/openai`
- **BitGN SDK**: `@buf/bitgn_api.connectrpc_es` + `@buf/bitgn_api.bufbuild_es` (ConnectRPC v1)
- **Schema**: Zod v4

## Key Pattern

```
Bootstrap (tree → AGENTS.md → context) → Task Instruction → Tool Calling Loop → report_completion
```

The agent uses **native tool calling** via `generateText()`, not `generateObject()`.
Claude doesn't support `oneOf`/`maxItems` in structured output, so native tool
calling is both more robust and plays to the model's strengths.

## Running

```bash
pnpm install
cp .env.local.example .env.local    # fill in BITGN_API_KEY + one LLM provider
pnpm start                          # all tasks, session mode
pnpm exec tsx src/main.ts t01       # single task
pnpm exec tsx src/main.ts --playground t01   # ad-hoc debug (not recorded)
```

## Environment

- `BITGN_API_KEY` — BitGN platform key (required)
- `ANTHROPIC_API_KEY` *or* `OPENAI_API_KEY` — at least one required
- `MODEL_ID` — defaults to `claude-sonnet-4-6`
- `BENCHMARK_ID` — `bitgn/pac1-dev` (practice) or `bitgn/pac1-prod` (scored)
- `ENABLE_SECURITY_SCAN` — injection scanning (default on)

Full list in [`.env.local.example`](.env.local.example).

## Scoring Priority

1. **Prompt injection resistance** — refuse malicious instructions from files
2. **Protocol compliance** — correct outcome enum + grounding refs
3. **Safe tool use** — minimal side effects, no unnecessary destructive actions
4. **Task completion** — correct side effects
5. **Speed** — fewer LLM calls = faster

## File Overview

| File | Purpose |
|------|---------|
| `src/main.ts` | Entry point — session/playground runner |
| `src/agent.ts` | Core agent loop (tool calling + dispatch + hardening) |
| `src/prompts.ts` | Soft-SGR system prompt with outcome decision tree |
| `src/schema.ts` | Zod tool schemas |
| `src/harness.ts` | BitGN `HarnessService` client (control plane) |
| `src/runtime.ts` | `PcmRuntime` client (tool dispatch) |
| `src/formatters.ts` | Tool output → shell-like rendering |
| `src/config.ts` | Environment config + feature flags |
| `src/hardening.ts` | Facade re-exporting defense layers B1–B5 |
| `src/paths.ts` | B1 — path-traversal guard |
| `src/pii.ts` | B2 — PII refusal detection |
| `src/refs.ts` | B3 — grounding-refs self-validation |
| `src/security.ts` | Injection + phishing scanner |
| `src/redaction.ts` | B5 — vendor-secret redaction |
| `src/messages.ts` | Sliding-window context pruning |
| `src/retry.ts` | Exponential backoff for transient errors |
| `src/metrics.ts` | Per-task Run-Metrics JSONL writer |
