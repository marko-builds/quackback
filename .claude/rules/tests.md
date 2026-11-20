---
paths:
  - '**/__tests__/**'
  - '**/*.test.ts'
  - '**/*.test.tsx'
---

# Tests

- Vitest, `globals: true`. A new test file goes in the `__tests__/` folder beside its module and is
  named after it in kebab case.
- Run the one file first: `bunx vitest run <path>`. If it fails to connect to Postgres, the suite
  needs the dev database (`bun run setup`, then `bun run db:migrate`).
- Assert on behaviour a caller can observe (the returned id, the thrown error type, the status
  code), not on which internal helper ran.
- When a fix changes a security property (a comparison, a scope check), add the case that must
  still be REFUSED next to the case that is now accepted. A fix that only adds the accepting case
  can pass while opening a hole.
- Never edit an existing assertion to make a new change pass without saying why in the PR.
