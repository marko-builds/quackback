# E2E quarantine

A quarantined test is one we know fails for a reason this suite cannot fix by
itself (missing infrastructure in CI, or a product question that needs an
answer). It stays in the code, so nobody loses it, but the nightly `e2e-full`
job leaves it out, so it cannot hide new failures.

## How it works

- Mark the test with the `@quarantine` tag and a `quarantine` annotation that
  gives the cause:

  ```ts
  test(
    'title',
    {
      tag: '@quarantine',
      annotation: { type: 'quarantine', description: 'Why it fails' },
    },
    async ({ page }) => {
      // ...
    }
  )
  ```

- `e2e-full` runs with `--grep-invert @quarantine`.
- `e2e-quarantine` runs nightly with `--grep @quarantine`. It never fails the
  workflow. Its job summary lists every quarantined test, its result and its
  cause, so a test that starts to pass is easy to see. It runs each test
  apart from the rest of its file, so one pass there does not prove a fix:
  run the whole file before you lift a quarantine.
- Every quarantined test has an entry below with an owner and a review date. On
  the review date, fix the test, fix the product, or delete the test with a
  reason. Do not extend the date without a reason in the entry.

## The rule: an owner and two weeks

- A test enters quarantine with a named owner (a person, not a team) and a
  review date no more than two weeks out. "owner: unassigned" is only allowed
  for the entries below that predate this rule, and each of them needs an owner
  before its review date.
- The owner either fixes it (test or product) and removes the tag, or deletes
  it with the reason in the commit, by the review date.
- One extension of up to two weeks is allowed, with the reason written in the
  entry. A second extension needs the code owner's sign-off in the PR.
- The nightly `e2e-quarantine` summary lists every quarantined test, so a test
  that has started to pass, or one past its date, is visible every morning.
- The support suite (`playwright.support.config.ts`) follows the same rule and
  tag; its nightly run leaves `@quarantine` out in the same way.

## Quarantined tests

Every entry has owner: unassigned and review by: 2026-12-02.

### Support email journeys (`tests/support/email-*.spec.ts`)

Known bugs (fixes in flight) and bugs found while writing these journeys. Each
test asserts the right behaviour; the working part of each journey is proved
by a separate test that is not quarantined.

### Support suite: tickets remainder

- `support/tickets-requester.spec.ts` › the ticket embed card › TKT-65 a
  ticket card shows its stage to the requester and is unavailable to another
  visitor. Product bug: a ticket embed (the embed-only message `shareTicket`
  sends, `conversation.cards.ts`) renders nothing in a thread, so the owner
  never sees the card and nobody else sees "This ticket is unavailable".
  `generateContentHTML` emits the embed placeholder only for `post` and
  `changelog` (`apps/web/src/lib/shared/content-html.ts:223`); a post embed
  stored the same way renders. In a local build patched to allow `ticket`
  there, the requester's thread then failed with "Something went wrong", so
  the fix likely needs more than that line. owner: unassigned, review by:
  2026-12-03.

### Support suite: workflows and the AI agent

- `support/automation-ai.spec.ts` › AI agent in the widget › WFL-76 the model
  closes the conversation. The `end_conversation` tool needs
  `conversation.set_status` (its spec in `assistant.toolspec.ts`), but the AI
  agent's permission ceiling (`ASSISTANT_PERMISSIONS`,
  `domains/assistant/assistant.actor.ts`) does not hold it, so the permission
  check in `assistant.tools.ts` denies every call ("This action is not
  permitted for the assistant.") and the conversation stays open. Runs in the
  support-isolated job. owner: unassigned, review by: 2026-12-03.

### Product suite: people sign-in and security (PPL-AUTH)

- `people/auth-portal.spec.ts` › PPL-AUTH-09 a reused or expired sign-in link
  says it expired. Product bug A-B1: the library redirects a spent or expired
  link with `error=INVALID_TOKEN` (uppercase), and the toast map only knows the
  lowercase `invalid_token` / `token_expired`
  (`apps/web/src/lib/shared/auth-block-messages.ts:74`,
  `apps/web/src/components/auth/use-auto-open-auth.ts:37`), so the person sees
  the generic "Sign-in failed. Try again or contact your administrator..."
  toast. Fix: map `INVALID_TOKEN` (and `EXPIRED_TOKEN`) to the expired-link
  copy. owner: unassigned, review by: 2026-12-03.
- `people/auth-portal.spec.ts` › PPL-AUTH-16 with the magic-link method off,
  portal-signin mails no working link. Product bug A-B2: `requestEmailSignin`
  (`apps/web/src/lib/server/auth/email-signin.ts:51`) and the route
  (`apps/web/src/routes/api/auth/portal-signin.ts:130`) never read
  `authConfig.oauth.magicLink`, and `/magic-link/verify` is never method-gated,
  so with the method off (the default) the endpoint still mails a link and code
  that sign anyone in. The settings row is called "Email magic link"; off means
  the method is not offered. Fix: refuse (or silently skip) the send when the
  method is off, keeping the uniform answer. Note: every other test that uses
  `/api/auth/portal-signin` on the default settings passes only because of this
  bug; once fixed, the suite baseline must turn the method on. owner:
  unassigned, review by: 2026-12-03.
