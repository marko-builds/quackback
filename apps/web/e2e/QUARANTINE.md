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

### Product suite: feedback admin (AI, notifications, import and export)

- `feedback/admin-import-export.spec.ts` › import route › FBA-X-12 a re-import
  with blank status and vote_count cells keeps the stored status and votes.
  Product bug: a `source_id` update writes whatever the row says, so a blank
  `status` resets the post to the default status and a blank `vote_count`
  sets it to 0 although votes may exist (`processBatch` update loop,
  `apps/web/src/lib/server/domains/import/import-service.ts:369`; the row
  schema turns a blank `vote_count` into 0 at `import-row-resolver.ts:58`).
  Fix: on the update path keep the stored `statusId`/`voteCount` when the
  cell is blank. owner: unassigned, review by: 2026-12-03.
- `feedback/admin-import-export.spec.ts` › import route › FBA-X-19 rows naming
  a deleted board, status or tag get fresh live ones. Product bug:
  `loadRowContext` reads `postStatuses`, `postTags` and `boards` with no
  `deleted_at` filter
  (`apps/web/src/lib/server/domains/import/import-row-resolver.ts:139`,
  `:148`, `:153`), so the post lands on a hidden board with a deleted status
  and tag. Fix: filter `isNull(deletedAt)` in the three lookups (and in
  `ensureBatchTag`). owner: unassigned, review by: 2026-12-03.
- `feedback/admin-import-export.spec.ts` › board export › FBA-X-02 a deleted
  board cannot be exported by id. Product bug: `/api/export` checks the board
  with `getBoardById`, which has no `deleted_at` filter
  (`apps/web/src/routes/api/export.ts:49`,
  `apps/web/src/lib/server/domains/boards/board.service.ts:321`), and the
  no-board branch of `listPostsForExport` lists every board
  (`apps/web/src/lib/server/domains/posts/post.export.ts:23`), so a deleted
  board's posts download. Fix: refuse a deleted board with `Board not found`
  and filter `deleted_at` in the all-boards list. owner: unassigned, review
  by: 2026-12-03.
- `feedback/admin-notifications.spec.ts` › FBA-N-15 the unread badge leaves out
  bells redacted because the post was deleted. Product bug: once a post is
  deleted, `getNotificationsForMember` redacts its bells (`Notification` /
  `This activity is no longer visible to you.`), but `getUnreadCount`
  (`apps/web/src/lib/server/domains/notifications/notification.service.ts:243`)
  counts every unread, unarchived row with no post visibility check, so the
  badge shows an unread count made of rows the person cannot open. Fix: apply
  the same post/board/moderation join (or the list's denial rule) in
  `getUnreadCount`, or exclude rows whose post is deleted or no longer visible.
  owner: unassigned, review by: 2026-12-03.
