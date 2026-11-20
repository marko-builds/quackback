---
name: prd-to-issues
description: Break a PRD, plan, or spec into independently-grabbable local markdown issues as vertical slices (tracer bullets), with blocking relationships (a dependency graph) and agent-ready vs needs-human tags. Use after a PRD is written, or when asked to "split this into issues", "break this into tickets/tasks", "make a backlog from this".
source-attribution: |
  Adapted from mattpocock/skills, skills/engineering/to-tickets (Matt Pocock).
  MIT licence, see LICENSE at the kit root.
---

# PRD to issues

Split a destination document into a backlog an agent can actually implement well. The AI defaults
to coding **horizontally** (all schema, then all API, then all UI) and gets no integrated feedback
until the end. Force **vertical slices** instead: each issue cuts through every layer and produces
something demoable or testable on its own.

## Process

### 1. Gather context

Work from the PRD or plan already in the conversation. If the user passes a path (e.g.
`prd-to-issues issues/prd-gamification.md`), read it fully first.

### 2. Explore the codebase (if not already)

Understand the current state. Issue titles and descriptions use the project's vocabulary (root
`CLAUDE.md`, path-scoped rules in `.claude/rules/`); respect the project's decision records (ADRs),
if it keeps any. Look for **prefactoring** that makes the implementation easier: "make the change
easy, then make the easy change." Any prefactoring becomes the first slice.

### 3. Draft vertical slices (tracer bullets)

Each slice is a thin, COMPLETE path through all integration layers (schema, service, API, UI,
tests), NOT a horizontal slice of one layer.

- A completed slice is demoable or verifiable on its own.
- Good: "award points for lesson completion, visible on the dashboard" (touches every layer).
- Bad: "create the gamification service" alone (a horizontal layer; push back on it).
- Watch yourself: even when told to slice vertically, the model drifts horizontal. Re-check.

### 4. Quiz the user, and run the agent-ready gate

Present the breakdown as a numbered list. Per slice show: **Title**, **Blocked by** (which slices
must finish first), **User stories covered**, **Type**, and **Verification**.

**The agent-ready gate: the metric IS the definition.** A slice is `agent-ready` only if its
correctness can be checked by an objective, cheap pass/fail signal the agent can score itself
against, drawn from a closed set: a failing **test** to turn green, a **typecheck**, a **build**, a
**benchmark**, an **eval**. No such signal means `needs-human` (a decision, credentials, a design
call). The gate is one-directional: the absence of a metric _is_ the answer, because agents are
trustworthy inside a verifiable domain and drift outside it. Front-end taste, irreversible data
operations, and ambiguous specs inherently lack a cheap signal, so they are human.

Test verifiability **in principle** here, not the finished assertion; authoring the actual test is
`tdd`'s job at implement time. So:

- Clear check available: `agent-ready`; Verification names the signal kind (`test`, `build`, ...).
- Fuzzy but a check is **manufacturable**: write the assertion first to make the behavior
  verifiable; tag `agent-ready`, Verification = that signal (or `TBD-by-tdd`). Prefer manufacturing
  a check over surrendering autonomy.
- No honest signal and none manufacturable: `needs-human`. Never invent a hand-wavy metric ("output
  looks right") to force an `agent-ready` tag. A fake green licenses an unguarded run and is worse
  than an honest `needs-human`.

**Coverage audit (against the PRD's Validation Contract, if it has one).** Every assertion in the
Validation Contract must appear in some slice's **Verification** line (by number). Before writing
issues, check the union of slices covers every assertion; flag any uncovered one: add a slice or
note why it is deferred. **Watch the needs-human gap:** if an assertion is covered _only_ by a
`needs-human` slice, it is asserted but never machine-checked. Surface it so the user confirms a
manual sign-off is acceptable rather than letting coverage read "green" with no real signal.

Ask: granularity right (too coarse or fine)? dependencies correct? merge or split any? **Is each
agent-ready slice's Verification a real signal, not theater? Does every contract assertion land in
some slice?** Iterate until the user approves.

### 5. Write the issues

For each approved slice, write `issues/NN-<slug>.md` (zero-padded order), or file it in the team's
issue tracker if CLAUDE.md names one. Use the template. Write in **dependency order** (blockers
first) so "Blocked by" can name real issue files. The blocking relationships make this a graph:
unblocked issues can be grabbed by independent agents or engineers in parallel, which is the
payoff over a sequential plan.

Do NOT close or modify the parent PRD.

**Write for the weakest executor.** The agent that implements an issue has zero context from this
conversation and no taste of its own; it may run hours later, unattended, on a smaller model. Four
rules follow, all cheap to obey now and expensive to skip:

1. **Stamp the commit.** Capture `git rev-parse --short HEAD` **once** before writing the batch and
   put it on every issue's `**Commit:**` line. It is the only way a later reader can tell whether
   the issue describes code that still exists.
2. **Name the boundaries, don't imply them.** Every issue gets a `## Boundaries` section listing
   what is explicitly out of scope: the adjacent files not to touch, the refactor not to start,
   whether new dependencies are allowed. An unbounded issue invites the agent to "improve" its way
   into an unrelated diff, which a per-issue reviewer is worst at catching.
3. **Split verification into mechanical and observable.** The header `**Verification:**` line names
   the contract assertion and the signal kind. The body `## Verification` section spells it out:
   **Mechanical** = the exact commands with their expected outcome; **Observable** = what a human
   or agent should actually see, for the part no command proves. A slice can be mechanically green
   and behaviourally wrong; the observable half is where that gets caught.
4. **Add the drift clause verbatim.** Every issue ends its Boundaries with: _"If the code you find
   doesn't match what this issue describes (drift since the Commit stamp above), STOP and report,
   do not improvise a fix."_ This is the single highest-value line in the template. An agent that
   improvises past a stale spec produces a confident, plausible, wrong diff and still reports
   success.

**One exemplar beats three paragraphs of convention.** Where an issue depends on a repo convention
(token file, test layout, naming, a wrapper the project always uses), point at **one existing
file:line that already does it right** and say "imitate this." Do not describe the convention in
prose: the description goes stale and the exemplar does not.

## Issue template

```markdown
# <NN>: <slice title>

**Status:** ready
**Type:** agent-ready | needs-human
**Parent:** issues/prd-<slug>.md
**Blocked by:** issues/NN-<slug>.md (or "None, can start immediately")
**Commit:** <short SHA at authoring time, from `git rev-parse --short HEAD`>
**Verification:** <names the PRD Validation-Contract assertion(s) this slice satisfies AND the
pass/fail signal that proves them, e.g. "assertions 3,4: failing test to turn green" | typecheck |
build | benchmark | eval | TBD-by-tdd for agent-ready slices; the manual check for needs-human.>

Keep Status, Type and Parent each on their OWN line (no joining with separators) so tooling and
people can read each field as "the rest of the line after the label". Keep the heading spelled
exactly `## Acceptance criteria`.

## What to build

Concise description of this vertical slice: the end-to-end behavior, not layer-by-layer
implementation. No file paths or code snippets (they go stale). Exception: a prototype snippet that
encodes a decision precisely (state machine, schema, type shape); inline just that, note its origin.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Repo conventions to follow

- <One existing `file:line` that already does this right: "imitate this.">
- <Only add a second line if a genuinely separate convention applies. Point, don't describe.>

## Verification

- **Mechanical:** <exact commands + expected outcome, e.g. `npm run typecheck` exits 0;
  `npx vitest run src/streak.test.ts` goes red to green. Never "run the tests".>
- **Observable:** <what a human or agent should actually SEE for the part no command proves, e.g.
  "the streak badge shows 3 after completing three days, and survives a cold restart".
  If a slice is fully machine-checkable, write "none, mechanical covers it" rather than
  inventing one.>
- **Done when:** <machine- or eye-checkable completion criteria>

## Boundaries

- Do NOT touch <files / components / layers out of scope for this slice>.
- Do NOT <start the adjacent refactor / rename / restructure this slice is not about>.
- Do NOT add new dependencies. <or: "the only new dependency permitted is X">
- **If the code you find doesn't match what this issue describes (drift since the Commit stamp
  above), STOP and report, do not improvise a fix.**
```

The last Boundaries bullet is **verbatim, on every issue**.

## When it's done

Offer to start implementing: take unblocked `agent-ready` issues one at a time with `tdd`, then
check each diff with `review-diff`. That step writes code, so confirm before kicking it off.

**Chain:** `grill-me`, then `write-prd`, then **prd-to-issues**, then `tdd`, then `review-diff`.
