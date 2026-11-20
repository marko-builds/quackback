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

### Product suite: feedback admin (merge, moderation and settings)

- `feedback/admin-merge.spec.ts` › FBA-M-11 accepting a merge suggestion
  needs post.merge, like merging by hand. Product bug: `acceptSuggestionFn`
  gates on `suggestion.manage` only (`apps/web/src/lib/server/functions/feedback.ts:33`)
  and calls `mergePost` (`merge-suggestion.service.ts:128`), so a role that
  holds suggestion.manage but not post.merge merges posts (mergePostFn refuses
  the same role). Fix: also require `PERMISSIONS.POST_MERGE` in
  `acceptSuggestionFn`. owner: unassigned, review by: 2026-12-03.
- `src/lib/server/domains/posts/__tests__/post-merge.db.test.ts` (vitest,
  `it.skip`) › FBA-M-12 a manual merge resolves the pending suggestions that
  name the merged post. Product bug: `mergePost`
  (`apps/web/src/lib/server/domains/posts/post.merge.ts:66`) never touches
  `post_merge_suggestions`; only `acceptMergeSuggestion` dismisses siblings
  (`merge-suggestion.service.ts:141`), so after a manual merge a suggestion
  naming the merged post stays pending (Similar posts card, inbox badge)
  until the 30-day expiry, and accepting it fails with ALREADY_MERGED. Fix:
  dismiss pending suggestions naming the duplicate inside `mergePost` (and on
  delete/reject). owner: unassigned, review by: 2026-12-03.
- `feedback/admin-moderation.spec.ts` › FBA-Q-04 a test customer held post is
  left out of the queue, as it is of the counts. Product bug: the sidebar
  badge, banner and filter-pane count exclude test customers
  (`apps/web/src/lib/server/functions/moderation.ts:133`, `notTestPrincipal`)
  but `listPendingPosts` / `listPendingComments`
  (`apps/web/src/lib/server/domains/moderation/moderation.service.ts:126`) do
  not, so the queue lists items the count leaves out. Fix: add
  `notTestPrincipal(posts.principalId)` (and the comment author's) to both
  list queries. owner: unassigned, review by: 2026-12-03.
- `feedback/admin-statuses-settings.spec.ts` › FBA-T-21 a status can be
  created again with the slug of a deleted one. `createStatus` looks the slug
  up among every row, soft-deleted ones included
  (`apps/web/src/lib/server/domains/statuses/status.service.ts:68`), so a
  deleted status blocks its slug for ever with `DUPLICATE_SLUG` for a status
  no page lists. Fix: add `isNull(postStatuses.deletedAt)` to that lookup.
  owner: unassigned, review by: 2026-12-03.
- `feedback/admin-statuses-settings.spec.ts` › FBA-T-25 marking another
  status default leaves exactly one default (`@isolated`). `updateStatus`
  with only `{ isDefault: true }` runs `setStatusAsDefaultAtomic`, then an
  UPDATE whose SET is empty
  (`apps/web/src/lib/server/domains/statuses/status.service.ts:139-149`);
  drizzle refuses it ("No values to set"), so the call answers 500 after the
  default already moved. Fix: skip the second UPDATE when `updateData` is
  empty and return the re-read row. owner: unassigned, review by: 2026-12-03.
- `feedback/admin-statuses-settings.spec.ts` › FBA-T-25 the only default
  status cannot be unset (`@isolated`). `updateStatus { isDefault: false }`
  on the default writes `is_default = false`
  (`apps/web/src/lib/server/domains/statuses/status.service.ts:142`), leaving
  no default (`getDefaultStatus()` null). Fix: refuse unsetting the default
  (`CANNOT_UNSET_DEFAULT`); a default only moves by marking another one.
  owner: unassigned, review by: 2026-12-03.
- `feedback/admin-tags-settings.spec.ts` › FBA-T-31 a tag can be created
  with the name of a deleted tag. The duplicate-name check spans
  soft-deleted tags (`apps/web/src/lib/server/domains/post-tags/post-tag.service.ts:53`),
  so the name of a deleted tag can never be reused and the error names a tag
  no page lists. Fix: add `isNull(postTags.deletedAt)` to the lookup (and to
  the same check in `updatePostTag`). owner: unassigned, review by:
  2026-12-03.
- `feedback/admin-tags-settings.spec.ts` › FBA-T-32 a deleted tag no longer
  shows on the posts it was on. `deletePostTag` only sets `deleted_at`
  (`apps/web/src/lib/server/domains/post-tags/post-tag.service.ts:148`);
  `post_tag_assignments` stay and the inbox and post queries load tags with
  no `deleted_at` filter (`apps/web/src/lib/server/domains/posts/post.inbox.ts:326`,
  `post.query.ts:105`), so the chip stays, while the confirm says the tag is
  removed from all posts. Fix: delete the tag's assignments in the same
  transaction (or filter deleted tags wherever post tags are read). owner:
  unassigned, review by: 2026-12-03.
