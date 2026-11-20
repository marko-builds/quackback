---
paths:
  - 'apps/web/src/lib/server/**'
---

# Server code

- Import rules from `apps/web/src/lib/README.md` hold: `server/` never imports `client/`, and
  `lib/` never imports `components/`. Database access goes through `@/lib/server/db`.
- A domain rule that fails throws a typed error from `@/lib/shared/errors`, so the API layer can
  map it to the right status (the `fix(api): answer 400, not 500` class of bug comes from a bare
  `Error` or an unchecked input reaching the handler).
- Permission checks live in the domain's `*.permissions.ts` or the shared `permissions.ts`; a new
  read path asks the same question the UI asks, never a looser one.
- Anything that verifies a secret (HMAC tags, tokens, signatures) compares with `timingSafeEqual`
  on equal-length buffers, and the length check comes first.
- Workspace scope: in pooled tenancy, `db` refuses a call with no workspace resolved. Code that runs
  outside a request (jobs, cron) must establish the workspace before touching `db`.
