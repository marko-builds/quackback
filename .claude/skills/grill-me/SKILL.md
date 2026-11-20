---
name: grill-me
description: A relentless interview, asked in rounds along a question graph, that stress-tests a plan, design, or vague idea until you and the AI share the same mental model, before any code or PRD gets written. Use when the user says "grill me", "stress-test this", "interview me", "let's align before building", or when about to plan something non-trivial and alignment is thin. Run this first, before planning or implementing.
source-attribution: |
  Adapted from mattpocock/skills (skills/productivity/grill-me and grilling).
  MIT licence, see LICENSE at the kit root.
---

# Grill me

The alignment step that comes before planning. The point is not to produce a plan, it is to
reach a **shared design concept**: you and the AI agree on what is being built, why, and what is
deliberately out of scope. Skipping it is the biggest source of AI rework (the model eagerly
builds the wrong thing). The conversation this produces is the asset: feed it straight into a PRD
afterward.

## The loop

Interview the user relentlessly about every aspect of the plan until you reach a shared
understanding. Walk down each branch of the decision tree, resolving dependencies between
decisions as they unblock.

**Facts vs decisions: the split that keeps this a real interview.**
A **fact** is something already true that you can find out yourself: what the schema holds, which
module owns a route, what an existing decision record settled. Go and find facts. Never ask the
user for one.
A **decision** is a choice that is not made yet, where the right answer depends on what the user
wants. Only the user makes decisions. Every question you ask must be one.
Without this split the session collapses inward: you start answering your own questions and call
the monologue an alignment. If you catch yourself proposing and accepting, you have drifted onto a
fact you should have looked up, or a decision you should have asked.

### Ask in rounds, along the question graph

Questions are a **graph, not a queue**. A few are critical and gate everything downstream ("is
this one screen or two?"); most only become askable once an earlier answer lands; some are
independent of each other and can be answered in any order.

- **A round is every question that is unblocked right now**: nothing whose answer depends on
  another question in the same round. Round 1 may be a single critical question. Later rounds are
  often five or six trivial ones.
- **Ask a whole round in one turn.** A round of six easy questions costs one reply instead of six
  turns. Never pad a round to look thorough; never include a question that depends on another one
  in the same round.
- **Number them `Q1`, `Q2`, and so on, and give every one a recommended answer** on its own line,
  prefixed `💡`. The user answers by number ("Q1 agree, Q2 agree, Q3 change X"), so the numbers
  and the 💡 are scan anchors. A question with a default is far cheaper than a blank one.
- **Push the frontier fast.** After each round, re-derive what just became askable and send the
  next round. State the round number (`## Round 3`).
- **Split the round when an answer would invalidate its siblings.** If Q2's answer could make Q3
  and Q4 meaningless, they were never unblocked: hold them for the next round.
- **A round of up to 4 pick-one questions goes through the AskUserQuestion tool** if your
  environment has it: recommended option first, labeled "(Recommended)". Rounds with free-form or
  5+ questions stay numbered text.

### Question quality

- **Go get the facts before you open your mouth.** Read the repo's root `CLAUDE.md`,
  `.claude/rules/`, and the project's decision records (ADRs), if it keeps any; only ask what the
  repo genuinely cannot tell you. Use a subagent for exploration so the main context stays lean.
- **If the project states its goals** (a product brief, a README section, an ADR), read them
  before Round 1: who it is for, the win condition, what is out of scope. Never re-ask those.
  Frame questions inside them, and when an answer would contradict them, say so rather than
  absorbing it silently.
- **Surface the nasty questions the user did not think to ask**: retroactive data, edge cases,
  migration and backfill, definition of done, what is explicitly not in scope. These need a human,
  which is why this phase cannot run unattended.
- **Keep going until the design concept is solid**, not until you have "enough for a plan."
  Sessions can run long (dozens of questions). If the user says pull back or wrap up, do.

## When it's done

**The gate: do not enact anything until the user confirms you have reached a shared
understanding.** Not when you judge the questions exhausted, but when they say so. Reaching the end
of your question list is the moment to state where you think you both landed and ask them to
confirm. Writing code, editing files, or opening a PRD before that confirmation defeats the
purpose of the session.

- Don't dump a plan unless asked: the deliverable is the aligned conversation itself.
- Offer the obvious next step: **"Write this up as a PRD?"** and **"Record any resolved
  decisions in the project's decision records (ADRs)?"**, especially out-of-scope calls and
  anything non-obvious you talked the user into or out of.

<supporting-info>
Reference material. The loop and the gate above are the instruction; nothing here overrides them.

**Inputs it works from:** a raw idea, a client or chat brief, a meeting transcript, a long voice
ramble, or "I want to build X." If the user opens with a ramble (possibly dictated, so expect
typos and incoherence), reconstruct the intent yourself, reflect a cleaned echo back, and open
Round 1 from it. Never ask the user to tidy the ramble.

**What a round looks like on the page:**

```
## Round 2

**Q1.** When a habit is archived mid-week, does its streak freeze or reset?
💡 Freeze. Resetting punishes a deliberate pause, and the model already handles a gap day.

**Q2.** Does the archived habit still render in the history view, greyed out?
💡 No, drop it entirely. A greyed-out state is a third visual mode to maintain.
```

The user replies "Q1 agree, Q2 keep it greyed out" and Round 3 follows immediately.

**Chain:** grill-me (align), then write-prd (destination), then prd-to-issues (vertical-slice
split), then tdd (implement), then review-diff (review in fresh context). This is the front of
that pipeline.
</supporting-info>
