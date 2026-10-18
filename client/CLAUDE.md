# client/ — `@devdigest/web`

Next.js 15 (App Router) · React 19 · TanStack Query 5 · next-intl 3 · Tailwind CSS 4 ·
recharts 2 · mermaid 11 · react-markdown 9 · Zod 3 · vitest 2 + jsdom + Testing Library 16. **pnpm 10.**

## Commands (run from `client/`)
- `pnpm dev` — web on :3000 (needs the API on :3001 for real data)
- `pnpm test` — vitest + jsdom, `fetch` mocked; no API or browser needed
- `pnpm typecheck` · `pnpm build`

## Conventions
- Data flows only `src/lib/hooks/*` → `src/lib/api.ts` (base `NEXT_PUBLIC_API_BASE`,
  default `http://localhost:3001`). Components never call `fetch` directly.
- Pages (`src/app/**/page.tsx`) stay thin; feature logic lives in colocated
  `_components/<Name>/` folders, each with its own `*.test.tsx`.
- UI building blocks come from the `@devdigest/ui` barrel (`src/vendor/ui`) — never deep-import
  a layer file. Check what exists there before writing a new component.
- User-facing copy goes in `messages/en/<namespace>.json` (next-intl), not inline strings.
- Path alias `@/*` → `src/*`.

## Gotchas
- `@devdigest/shared` here resolves to `src/vendor/shared` — a **copy** of
  `server/src/vendor/shared` that has already drifted. Server is canonical: mirror edits
  from there; don't evolve the client copy on its own.
- `client.yml` CI watches only `client/**` — a server contract change won't run client tests;
  run `pnpm typecheck` here yourself after mirroring a contract.
- Unit tests mock `fetch`; real browser journeys belong in `../e2e`, not here.

## Do not touch
- `src/vendor/shared/**` — only mirror changes from the server copy (see above).

## Reference docs
- `README.md` — UI route map and the API endpoints each route uses.
  **Read when:** adding a page, route, or data hook.
- `src/vendor/ui/README.md` — design-system layers, tokens, import rules.
  **Read when:** building or restyling any UI.
- `../TESTING.md` — **Read when:** adding tests or deciding unit vs e2e coverage.
- `docs/README.md` · `specs/README.md` — **Read when:** designing or implementing a UI feature.
- `INSIGHTS.md` — **Read when:** debugging or changing non-trivial client behaviour.
