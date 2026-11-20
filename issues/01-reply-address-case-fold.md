# 01: a reply address in any letter case threads into its conversation or ticket

**Status:** ready
**Type:** agent-ready
**Parent:** issues/prd-reply-address-case.md
**Blocked by:** None, can start immediately
**Commit:** 2dbeefe34
**Verification:** assertions 1-5: failing tests in conversation-email-channel.test.ts turn green; lint exits 0

## What to build

When an inbound reply address comes back with its local part lowercased or uppercased by the
customer's mail client, the reader still resolves it to the conversation or ticket it was minted
for. The signed tag is compared without regard to letter case, still in constant time over
equal-length buffers, and the TypeID suffix is read in lower case. An address with a replaced tag
resolves to nothing in any case. Minting does not change. The comment above the address regex that
says lower-casing would fail every verification is corrected to say why case is now folded.

## Acceptance criteria

- [ ] A lowercased conversation reply address resolves to the original conversation id (assertion 1)
- [ ] A lowercased ticket reply address resolves to the original ticket id (assertion 2)
- [ ] An uppercased conversation reply address resolves to the original conversation id (assertion 3)
- [ ] A tampered tag is refused as minted, lowercased and uppercased (assertion 4)
- [ ] The existing round-trip and display-name tests still pass unchanged (assertion 5)

## Repo conventions to follow

- `apps/web/src/lib/server/domains/conversation/__tests__/conversation-email-channel.test.ts:303`
  ("reads a slug back case-insensitively"): imitate this for the new cases, in the same describe block.

## Verification

- **Mechanical:** `bunx vitest run apps/web/src/lib/server/domains/conversation/__tests__/conversation-email-channel.test.ts`
  goes red (new cases) then green, 84 existing tests still passing; `bun run lint` exits 0;
  `bun run --filter @quackback/web typecheck` adds no error in `conversation.email-channel.ts`
  (compare against the error list before the change: this checkout's baseline is not clean).
- **Observable:** none, mechanical covers it (the readers are pure functions).
- **Done when:** all five criteria are ticked and the diff touches only the two files named above.

## Boundaries

- Do NOT touch minting (`inboundAddress`, `signInboundTag`) or the alphabet.
- Do NOT touch `conversation.email-inbound.service.ts` or the `In-Reply-To` fallback.
- Do NOT add new dependencies.
- **If the code you find doesn't match what this issue describes (drift since the Commit stamp
  above), STOP and report, do not improvise a fix.**
