# e2e/ — `@devdigest/e2e`

Deterministic browser flows driven by the `agent-browser` CLI (Rust + CDP), run by `run.ts`
(tsx). TypeScript 5.7. **npm.** No Playwright, no LLM, no API key.

## Commands
- Once: `npm i -g agent-browser && agent-browser install`
- `./scripts/e2e.sh` (from repo root) — **default**: isolated, freshly seeded stack on
  Postgres :5433 / API :3101 / web :3100, torn down afterwards
- `npm test` (from `e2e/`) — against your running stack; only safe if the dev DB holds
  just the seeded repo
- `npm run typecheck`

## Conventions
- `specs/` here = **executable flows**: `specs/NN-name.flow.json` (run.ts picks up only
  `*.flow.json`). New flow → next free `NN` prefix. Markdown design notes go to `docs/`.
- Each step's `cmd` is passed verbatim to agent-browser; a non-zero exit fails the flow, so
  `wait --text` / `wait --url` are the assertions. Optional `assert.stdoutIncludes`.
- `{BASE}` is replaced with `E2E_BASE_URL` (default `http://localhost:3000`).

## Rules
- Deterministic locators only: `--url`, `--text`, `find role|text|label`. Never use `chat`.
- Flows target read-only seeded data (`acme/payments-api`, PR #482, seeded agents). Never
  submit forms or trigger anything that calls a model.

## Gotchas
- Flow 02 follows the home redirect to the *first* repo — against a dev DB with other repos,
  flows 02/04/05 fail. Use `./scripts/e2e.sh`, don't "fix" the flows.
- Failure screenshots land in `test-results/` (git-ignored; uploaded as a CI artifact).
- IMPORTANT: never `docker compose down -v` to get a clean DB — use the hermetic runner.

## Reference docs
- `README.md` — flow format, env knobs, coverage table.
  **Read when:** adding or changing a flow, or a run fails unexpectedly.
- `../TESTING.md` — **Read when:** deciding whether something belongs in e2e or a unit test.
- `docs/README.md` — **Read when:** planning e2e coverage or changing the runner.
- `INSIGHTS.md` — **Read when:** a flow is flaky or debugging the runner.
