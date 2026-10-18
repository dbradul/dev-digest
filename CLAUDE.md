<!-- Maintainers (HTML comments are stripped before Claude sees this file):
- This file is a map, not documentation. Line test: "remove it — will Claude err?" No → cut.
- Budget: root ≤100 lines, module CLAUDE.md ≤60. README.md / docs/ stay the source of truth.
- No @import yet — plain paths only (lazy, read on demand). Wrap every `@scope/pkg` in
  backticks: a bare @… outside a code span is parsed as an import.
- Module CLAUDE.md auto-loads in the CLI when Claude touches that folder; the VS Code
  extension doesn't (anthropics/claude-code#24987) → keep the explicit pointers in Map.
-->
# DevDigest

Local-first AI pull-request review: import a PR, run an agent review, get grounded findings.
The server clones and indexes a repo (repo-intel), `reviewer-core` turns diff + repo map into
an LLM prompt, and every finding must cite a real diff line or it is dropped.

## Map — read the module's CLAUDE.md before editing in it
Four standalone packages, **no workspace**: each has its own package.json + lockfile;
cross-package code is shared only through tsconfig path aliases.

| Path             | What                                                  | PM   | Port |
|------------------|-------------------------------------------------------|------|------|
| `server/`        | Fastify 5 API + Drizzle/Postgres (pgvector) → `server/CLAUDE.md` | pnpm | 3001 |
| `client/`        | Next.js 15 / React 19 studio → `client/CLAUDE.md`     | pnpm | 3000 |
| `reviewer-core/` | pure review engine → `reviewer-core/CLAUDE.md`        | npm  | —    |
| `e2e/`           | deterministic agent-browser flows → `e2e/CLAUDE.md`   | npm  | —    |
| `docs/`          | cross-cutting docs (agent prompts)                    | —    | —    |

Every module has: `README.md` (what/how) · `docs/` (design notes) · `specs/` (feature specs;
in `e2e/` = executable flows) · `INSIGHTS.md` (lessons learned).

## Stack
Node ≥22 · pnpm **10** (pnpm 12 fails install on unapproved build scripts) · TypeScript 5.7 ·
vitest 2 · Zod 3 · Postgres 16 + pgvector in Docker (only the DB runs in Docker).

## Commands
- `./scripts/dev.sh` — boot from zero: Postgres → deps → migrate → seed → API + web.
  Flags: `--no-seed` `--no-client` `--db-only`.
- `./scripts/e2e.sh` — hermetic e2e on alternate ports; never touches the dev DB.
- Per-package test / typecheck commands → that module's CLAUDE.md.

## Cross-cutting rules
- `@devdigest/shared` (Zod contracts): **canonical copy is `server/src/vendor/shared`**
  (server + reviewer-core alias to it). `client/src/vendor/shared` is a separate copy that
  has already drifted — when you change a contract the client uses, mirror the edit there.
  Extend contracts with new files/fields; don't rewrite existing ones.
- CI is path-filtered per package (`.github/workflows/*.yml` `paths:`). Adding a
  cross-package import or alias → add that path to the consumer's workflow filter.
- Secrets never go in git, the DB, or `AppConfig` — only `~/.devdigest/secrets.json` via
  `LocalSecretsProvider` (server), with `process.env` as fallback.
- IMPORTANT: never run `docker compose down -v` — it deletes the `devdigest_pgdata` volume
  with every imported repo and review.

## Reference docs — read before acting, don't guess
- `README.md#architecture` — packages and the end-to-end review flow.
  **Read when:** a change spans more than one package.
- `TESTING.md` — suite map, unit/integration split, CI path filters.
  **Read when:** adding a test or touching `.github/workflows/`.
- `docs/agent-prompts/README.md` — how prompts are assembled; output schema lives out of band.
  **Read when:** editing a reviewer system prompt or the `Review` / `Finding` contract.
- `<module>/specs/README.md` — specs index for that module.
  **Read when:** starting a feature or course lesson; reuse an existing spec before writing one.
- `<module>/INSIGHTS.md` — lessons learned the hard way.
  **Read when:** debugging, or changing non-trivial behaviour in that module.

## Keeping docs current
- Learned something non-obvious (a trap, a failed approach)? Add a dated entry to that
  module's `INSIGHTS.md` — not here.
- Changed behaviour a README describes? Update the README in the same change.
