# PRD: reply-by-email survives a mail client that changes the address case

**Status:** ready
**Project:** apps/web, conversation email channel
**Date:** 2026-11-20
**Upstream issue:** QuackbackIO/quackback#818

## Problem Statement

A customer replies to a support email from Yahoo Mail. Yahoo lowercases the local part of the
reply address, so the signed tag in `reply+c<id>.<tag>@` no longer matches, and the reply opens a
new cold conversation instead of threading into the original. The `In-Reply-To` fallback cannot
rescue it when the outbound provider rewrites the Message-ID (Amazon SES does). The team sees a
duplicate conversation with no history, and the customer's context is lost.

## Solution

A reply whose address arrived in a different letter case still threads into the conversation (or
ticket) it answers. A forged or tampered address is still refused, whatever its case.

## User Stories

1. As a customer on Yahoo Mail, I want my reply to land in the conversation I am answering, so that I do not have to repeat myself.
2. As a support agent, I want a customer's reply to appear in the existing thread, so that I see the history and do not answer a duplicate.
3. As a customer replying to a ticket email, I want the same, so that ticket replies thread like conversation replies.
4. As a workspace owner, I want an address with a guessed or altered tag to keep being refused, so that nobody can post into a conversation they were never sent.
5. As a customer whose mail client uppercases the address, I want my reply to thread too, so that the fix does not depend on which direction the client folds.
6. As a self-hoster with mail already in customers' inboxes, I want every reply address minted before the fix to keep working, so that upgrading changes nothing for old threads.

## Implementation Decisions

- Verification folds letter case on both the provided and the expected tag before the
  constant-time comparison. Minting is unchanged: tags stay base64url, so every address already
  sent keeps verifying.
- The TypeID suffix read from the address is folded to lower case too (minted suffixes are lower
  case), which covers a client that uppercases the local part. The slug is already folded today.
- Both reply families (conversation and ticket) go through the one shared verification function,
  so the fix lands once.
- Security: folding case maps the 64-symbol base64url alphabet onto 38 symbols. A 22-character
  tag goes from about 132 bits to about 115, still far past forgeable. The comparison stays
  `timingSafeEqual` over equal-length buffers.
- The comment at the address regex that says lower-casing "would fail every verification" becomes
  false and is corrected in the same change.

## Validation Contract

### Assertions

1. A conversation reply address with its whole local part lowercased resolves to the original conversation.
2. The same holds for a ticket reply address.
3. A reply address with its whole local part uppercased resolves to the original conversation.
4. An address whose tag was replaced, in any letter case, resolves to nothing.
5. An address exactly as minted still resolves, display-name wrapper included (existing behaviour).

### Test strategy

- One seam: the exported readers `conversationIdFromInboundAddress` and
  `ticketIdFromInboundAddress`, through the existing suite
  `conversation/__tests__/conversation-email-channel.test.ts` (pure, no database).
- Prior art in the same file: "reads a slug back case-insensitively" and "rejects a tampered tag".
  The new cases sit beside them in the same shape.
- Assertion 4 is the guard: it must be written so it would fail against a verifier that skipped
  the comparison entirely.

## Out of Scope

- Changing the minting alphabet to a case-proof one (base32 or hex). Decided against in the grill:
  two formats to maintain and a bigger diff for the same user outcome.
- The `In-Reply-To` / `References` fallback and Message-ID rewriting by SES.
- Any part of the agent setup on this fork. The upstream PR carries only this fix.

## Further Notes

- Grill decisions (2026-11-20): verify case-insensitively; fold the tag and the TypeID suffix;
  PR contains only the fix and closes #818.
