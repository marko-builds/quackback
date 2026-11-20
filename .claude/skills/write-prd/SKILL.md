---
name: write-prd
description: Synthesize the current conversation into a PRD (the destination document): problem, solution, user stories, implementation and testing decisions, out-of-scope, written as a local markdown file in the repo. Use after a grill-me alignment session, or when the user says "write a PRD", "turn this into a spec", "write up what we decided". Synthesis only, it does not interview (grill first if alignment is thin).
source-attribution: |
  Adapted from mattpocock/skills (skills/engineering/to-spec).
  MIT licence, see LICENSE at the kit root.
---

# Write a PRD

Turns an aligned conversation into the **destination document**: a written record of where the
team is going, dense enough to drive implementation. It does not interview, it synthesizes what you
already discussed. If alignment is thin, run grill-me first; the grilling history is the ideal
input.

The PRD's only job is to capture the shared design concept. Don't over-polish it (you won't
re-read it, you'll summarize it): the real work is QA later. Optimizing a PRD past "good enough"
yields little; alignment and QA are where quality lives.

## Process

1. **Ground in the repo.** Explore the project you are in: its root `CLAUDE.md`,
   `.claude/rules/`, and the project's decision records (ADRs), if it keeps any. Use the repo's own
   vocabulary throughout the PRD. Use a subagent for exploration if the codebase is large, to keep
   the main context lean. If the project states its goals (a product brief, a README section),
   read them first, cite a verbatim span on the PRD's `Serves:` line, and make sure Out of Scope
   does not contradict them. A PRD that contradicts the stated goals is a proposal to change those
   goals, not a PRD.

2. **Sketch the test seams, then the assertions.** Name the seam(s) at which the feature will be
   tested. **Prefer existing seams; prefer the highest seam possible; fewer seams is better, ideal
   is one.** This pushes the work toward deep, testable modules. Then draft the **Validation
   Contract** assertions (below): the behaviors that must hold, independent of how they are built.
   The seams are where you will check them, the assertions are what must be true. Asserting
   behavior up front, before code, is what stops an agent from later writing tests that merely
   rubber-stamp the code it wrote. Check with the user that the seams and assertions match their
   expectations before writing the body.

3. **Write the PRD** using the template, then save it as `issues/prd-<slug>.md` (create `issues/`
   if missing), or publish it to the team's issue tracker if `CLAUDE.md` names one. Add
   `**Status:** ready` at the top.

## PRD template

```markdown
# PRD: <feature>

**Status:** ready
**Project:** <project or area>
**Date:** <YYYY-MM-DD>
**Serves:** "<a verbatim span of the project's stated goals, 12+ characters; omit the line when there are none>"

## Problem Statement

The problem the user faces, from the user's perspective.

## Solution

The solution, from the user's perspective.

## User Stories

A LONG, numbered list. Each: "As an <actor>, I want <feature>, so that <benefit>."
Be extensive: cover every aspect of the feature.

## Implementation Decisions

- Modules to build or modify and the interfaces of those modules
- Architectural decisions, schema changes, API contracts, specific interactions
- Technical clarifications from the user
  Do NOT include file paths or code snippets (they go stale).
  Exception: if a prototype produced a snippet that encodes a decision more precisely than prose
  (state machine, reducer, schema, type shape), inline just the decision-rich bit and note it came
  from a prototype.

## Validation Contract

The definition of "done," written now, before any code, as correctness assertions independent of
implementation.

### Assertions

A numbered list of observable behaviors, one line each, phrased about external behavior not code
("a user who completes a lesson sees their point total rise on the dashboard"). **Keep it small on
purpose: 3-8 for a typical feature; if you pass about 12, the PRD is too big, so split it.** Each
assertion becomes a verification target a slice must satisfy in prd-to-issues.

### Test strategy

- What makes a good test here (test external behavior, not implementation details)
- Which modules will be tested; prior art (similar tests already in the codebase)

## Out of Scope

What this PRD deliberately does NOT tackle. This is part of the definition of done, and it
preserves the "things we decided against" from the grilling session. Don't skip it.

## Further Notes

Anything else.
```

## Pitfalls

- **When the plan's centre is a quantitative model, write the executable sim first.** Prose
  arithmetic reads as verified when it isn't: a hand-checked economy looked fine until a
  simulation showed the clock actually grew past a threshold. Write the model as a script, solve
  the constants from it, and cite the script as the source of truth in the PRD.
- **Never write a "dependency order" section.** A pure layer-cake ("domain layer, then service,
  then UI") makes prd-to-issues override it later. Name the build order by user-visible behaviour
  ("the timer runs out and you fail"), and let each domain class arrive with the slice that needs
  it. The horizontal drift the slicing step warns about starts one document upstream.
- **Per-document approval is not design approval.** When a feature spans several specs, each
  approved alone, mutually exclusive assumptions can live between them. After any per-file pass,
  do one explicit cross-document contradiction sweep over the shared nouns before calling a spec
  buildable.

## When it's done

Offer the next link: **"Split this into vertical-slice issues?"** (prd-to-issues). Don't auto-run
it. Don't review the PRD line by line on the user's behalf: if you reached alignment by grilling,
you are only checking the model's summarization, which it is reliably good at.

**Chain:** grill-me (align), then **write-prd** (destination), then prd-to-issues (split), then
implement.
