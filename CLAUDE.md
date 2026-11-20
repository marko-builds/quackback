# Quackback

Open-source customer feedback platform: feedback boards, roadmaps, changelog, help center, a support
inbox with email and widget channels, and an embeddable widget. TypeScript monorepo on Bun.
AGPL-3.0; outside contributions need the CLA (`CLA.md`).

Read `CONTRIBUTING.md` (architecture, setup) and `apps/web/src/lib/README.md` (server/client/shared
split, import rules, domain layout) before a change that crosses a module boundary. This file does
not repeat them.

## Commands

| Need               | Command                                                                    | Needs Postgres?     |
| ------------------ | -------------------------------------------------------------------------- | ------------------- |
| Install            | `bun install --frozen-lockfile`                                            | no                  |
| One test file      | `bunx vitest run <path/to/file.test.ts>`                                   | depends on the file |
| All unit tests     | `bun run test --run`                                                       | yes                 |
| Lint               | `bun run lint` (oxlint)                                                    | no                  |
| Typecheck the app  | `bun run --filter @quackback/web typecheck`                                | no                  |
| Widget tests       | `bun run --cwd packages/widget test`                                       | no                  |
| Migrations + drift | `bun run db:migrate`, then `bun run db:check-drift`                        | yes                 |
| Full dev stack     | `bun run setup` once (Docker: Postgres, Silo, Mailpit), then `bun run dev` | yes                 |

- CI (`.github/workflows/ci.yml`) runs lint, build, typecheck and four sharded unit-test jobs after
  `db:migrate`. Run lint, typecheck and the touched test files before calling a change done.
- **Use bun 1.4.0**, the `packageManager` pin CI uses. Installed with bun 1.4.4, two copies of
  `drizzle-orm` get linked and typecheck reports about 11,600 errors in files nobody touched;
  under 1.4.0 the same commit typechecks clean. With mise: `mise exec bun@1.4.0 -- bun install`.
- Typecheck needs the generated route tree: run `bun run build` once on a fresh checkout.
- Many unit suites talk to Postgres (`DATABASE_URL` defaults to `quackback_test` on localhost in
  `vitest.config.ts`). Pure-function suites do not: `conversation-email-channel.test.ts` runs 84
  tests with no database. Try the file alone first; start Docker only when it fails to connect.
- `*-integration.test.ts` and `e2e/` are excluded from `bun run test`; e2e runs on Playwright.
- The pre-commit hook runs `oxlint --fix` and `prettier --write` on staged files (lint-staged).

## Where things live

- `apps/web/src/lib/server/domains/<feature>/`: business logic, one folder per domain (56 of them).
  Services throw the typed errors from `@/lib/shared/errors` (`ValidationError`, `NotFoundError`,
  `ForbiddenError`, `ConflictError`), never bare `Error` for a domain rule.
- `apps/web/src/lib/server/functions/`: TanStack `createServerFn` endpoints, thin over the domains.
- `apps/web/src/routes/`: file-based routes. `routeTree.gen.ts` is generated; never edit it.
- `packages/db`: Drizzle schema and migrations. App code imports `@/lib/server/db`, never
  `@quackback/db` directly.
- Tests sit beside the code in `__tests__/`, named after the module in kebab case
  (`conversation.email-channel.ts` is tested by `__tests__/conversation-email-channel.test.ts`).
- `.impeccable.md`: design context for any UI change (users, tone, palette rules).

## Conventions the code already follows

- **Comments explain why, at length, and cite the issue** (`(#293)`). A rule with a non-obvious
  reason gets the reason next to it. Match this when you add a branch whose purpose is not
  visible from the code; do not add comments that narrate what the next line does.
- **Commit and PR titles:** `type(scope): what the user now sees, in plain words (#issue)`, e.g.
  `fix(api): answer 400, not 500, for bad request input`. Types in use, by frequency over the
  last 500 commits: `fix`, `feat`, `test`, `refactor`, `perf`, `chore`, `docs`.
- File names kebab-case, components PascalCase, functions camelCase, tables snake_case.
- Services stay under about 400 lines; split by responsibility rather than growing one file.
- Security-sensitive comparisons use `timingSafeEqual` on equal-length buffers; keep that shape.

## How work moves here (the agent chain)

Skills live in `.claude/skills/`. For anything bigger than a one-file fix:

1. `grill-me`: align on the problem before planning. Ends when the open questions are answered.
2. `write-prd`: the aligned conversation becomes `issues/prd-<slug>.md`.
3. `prd-to-issues`: vertical-slice issues in `issues/NN-<slug>.md`, each tagged `agent-ready` or
   `needs-human`.
4. `tdd`: one failing test, then the code that passes it, one behaviour at a time.
5. `review-diff`: a standards pass and a spec pass on the diff before the PR.

Upstream work is tracked in GitHub Issues on `QuackbackIO/quackback`. The local `issues/` files are
the working notes for a change; the PR description links the GitHub issue it closes.
