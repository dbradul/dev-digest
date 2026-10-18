# reviewer-core/ — `@devdigest/reviewer-core`

Pure review engine: diff → prompt → LLM → structured output → grounded findings.
TypeScript 5.7 · `openai` SDK 4 (OpenRouter-compatible) · Zod 3 · vitest 2. **npm, not pnpm.**

## Commands (run from `reviewer-core/`)
- `npm test` — hermetic, stubbed `LLMProvider`; no keys, no network
- `npm run typecheck` — this *is* the build; the package never emits JS
- After a change, also run `pnpm typecheck` in `../server` (it compiles this source directly).

## Conventions
- The server consumes `src/` as TypeScript source via a tsconfig alias — the public surface is
  `src/index.ts`; export new API from there.
- Contracts (`Review`, `Finding`, `Verdict`, …) come from `@devdigest/shared`, aliased to
  `../server/src/vendor/shared`. There is no local copy — change contracts there.
- The LLM is always injected (`LLMProvider`); `src/review/run.ts` orchestrates a run.

## Gotchas
- `server-unit` CI also triggers on `reviewer-core/**`, and `reviewer-core.yml` triggers on
  `server/src/vendor/shared/**` — a change here can turn the server's CI red.
- `verdict` is passed through from the model; `score` is recomputed from grounded findings.
- The output schema is sent out of band (`response_format` json_schema, strict), not in prompt text.

## Do not touch
- IMPORTANT: stay pure — no DB, filesystem, GitHub, or network access except through the
  injected `LLMProvider`.
- `INJECTION_GUARD` / `wrapUntrusted` (`src/prompt.ts`): don't weaken, and don't replace with
  keyword scanning of untrusted text.
- The grounding gate (`src/grounding.ts`) and score recomputation — findings without a real
  diff line must stay dropped; never trust the model's self-reported score.
- Prompt slots `skills` / `memory` / `specs` / `callers`, `reduce()` and `toReview()` look
  unused in the starter — later course lessons feed them; don't remove them as dead code.

## Reference docs
- `README.md` — pipeline diagram and public API.
  **Read when:** changing the pipeline or the exported surface.
- `../docs/agent-prompts/README.md` — message assembly, severity/verdict conventions.
  **Read when:** touching `src/prompt.ts`, structured output, or scoring.
- `../TESTING.md` — **Read when:** adding tests.
- `docs/README.md` · `specs/README.md` — **Read when:** designing or implementing engine changes.
- `INSIGHTS.md` — **Read when:** debugging review output or changing engine behaviour.
