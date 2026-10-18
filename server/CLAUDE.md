# server/ — `@devdigest/api`

Fastify 5 · Drizzle ORM 0.38 (+ drizzle-kit 0.30) over `postgres` · Postgres 16 + pgvector ·
Zod 3 via `fastify-type-provider-zod` 4 · vitest 2 + testcontainers 10 · tsx. **pnpm 10.**

## Commands (run from `server/`)
- `pnpm dev` — API on :3001 (tsx watch) · `pnpm typecheck`
- `pnpm db:generate` (after schema change) · `pnpm db:migrate` · `pnpm db:seed` (idempotent)
- Unit (no Docker): `pnpm exec vitest run --exclude '**/*.it.test.ts'`
- Integration (Docker): `pnpm exec vitest run .it.test` · both: `pnpm test`
- There are no `test:unit` / `test:integration` scripts — use the `pnpm exec` forms above.

## Conventions
- One feature = one plugin in `src/modules/<name>/` (`routes.ts` + service), registered
  statically in `src/modules/index.ts`.
- Validate with zod `params`/`body` schemas on the route; never `Schema.parse(req.body)`
  in a handler. Errors: throw `AppError` (`src/platform/errors.ts`) → structured envelope.
- All outside I/O (LLM, GitHub, git, ast-grep, secrets…) goes through adapters resolved
  from the DI container (`src/platform/container.ts`). Tests use `src/adapters/mocks.ts`.
- Config via `loadConfig` (`src/platform/config.ts`); secrets are not part of `AppConfig`.
- Stable prompt instruction text lives in `src/prompts/*.md` (loaded by `src/platform/prompts.ts`).

## Gotchas
- Migrations are **not** applied on boot — `relation … does not exist` means run `pnpm db:migrate`.
- A test that imports `test/helpers/pg.ts` **must** be named `*.it.test.ts`, or the unit lane breaks.
- The server runs `../reviewer-core/src` as source via alias: `reviewer-core/node_modules`
  must be installed, and a reviewer-core change can break `server-unit` CI.
- `src/platform/{grounding,prompt,structured}.ts` are re-export shims — change the logic
  in `reviewer-core/`, not here.
- Repo-intel is on by default, but an unindexed repo silently degrades to diff-only context.
- Global rate limit is disabled under `NODE_ENV=test`; don't "fix" tests by tuning it.

## Do not touch
- IMPORTANT: never edit an existing file in `src/db/migrations/` — change `src/db/schema*`
  and run `pnpm db:generate` to add a new migration.
- Tables that look unused — the schema already holds every course-lesson table; don't drop them.
- `clones/` — runtime checkouts of imported repos (git-ignored).
- `src/vendor/shared` — canonical contracts; extend, don't rewrite (see root CLAUDE.md).

## Reference docs
- `README.md` — request/DI flow, API map, env vars.
  **Read when:** adding a route, module, adapter, or env var.
- `README.md#review-context-non-obvious` — what actually reaches the model.
  **Read when:** touching `src/modules/reviews/run-executor.ts` or review inputs.
- `../TESTING.md` — **Read when:** adding a test or changing the unit/integration split.
- `docs/README.md` · `specs/README.md` — **Read when:** designing or implementing a server feature.
- `INSIGHTS.md` — **Read when:** debugging or changing non-trivial server behaviour.
