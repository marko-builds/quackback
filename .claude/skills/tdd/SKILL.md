---
name: tdd
description: Test-driven development done as vertical tracer bullets (one test, one implementation, repeat), the way that works with AI agents. Use when building a feature or fixing a bug test-first, when asked for "red-green", "TDD", or "write a failing test first", or when an unattended agent needs a discipline that stops it cheating its tests.
source-attribution: |
  Adapted from mattpocock/skills, skills/engineering/tdd (Matt Pocock).
  MIT licence, see LICENSE at the kit root.
---

# Test-driven development

**This is reference material, not a procedure with gates.** It specifies the _order_ you write in
(red before green, one slice at a time) and nothing that requires a human. An unattended agent can
follow the whole thing without stopping to ask. Never insert a confirmation step here: this skill
may be handed to agents that have nobody to confirm with, and a gate they cannot satisfy either
stalls the run or gets answered by invention.

## Anti-pattern: horizontal slicing

**DO NOT write all tests first, then all implementation.** Tests written in bulk test _imagined_
behavior: they end up asserting the _shape_ of things (data structures, signatures), pass when
behavior breaks, and fail when it is fine.

```
WRONG (horizontal):  RED: test1..test5   then   GREEN: impl1..impl5
RIGHT (vertical):    RED then GREEN: test1 + impl1, then test2 + impl2, ...
```

Each cycle responds to what you learned from the last.

## Workflow

### 1. Plan

- Read the repo's root `CLAUDE.md` (and any path-scoped rules in `.claude/rules/`) so test names and
  interface vocabulary match the domain; respect the project's decision records (ADRs), if it keeps
  any.
- Derive the interface changes and **which behaviors matter most** from the issue file, spec or PRD
  you were given. You can't test everything, so focus on critical paths and complex logic. The
  originating document is the authority; don't widen scope past it.
- If a behavior is genuinely ambiguous: with a person present, ask in one line and keep going;
  unattended, take the **narrowest** reading that satisfies the issue and record the assumption in
  your handoff notes (an "Issues discovered" line). Never stall waiting on a human.
- Spot deep-module opportunities (small interface, deep implementation) and prefer them when they
  make the code easier to test.

### 2. Tracer bullet

Write ONE test for ONE behavior. **RED** (it fails, e.g. the module doesn't exist) then **GREEN**
(minimal code to pass). Proves the path end-to-end.

### 3. Incremental loop

For each remaining behavior: RED (next test fails) then GREEN (minimal code to pass). One test at a
time; only enough code to pass the current test; don't anticipate future tests; keep tests on
observable behavior.

**Before calling the whole slice green, run the full test command once.** If the suite shows ANY
red you did not just write, find out whether it predates your diff: run the same command on the
base commit (a clean checkout or `git worktree`, not by stashing your work) and compare the lists of
failing tests. A red test that predates the diff is not this slice's to fix (note it and move on); a
red test the diff introduced is. Never eyeball a failing-test list and guess which case you are in.

### 4. Tidy: NOT a third beat of the loop

The loop is **red then green**, not red-green-refactor. Restructuring mixed into an implement
session overloads it: the diff stops being traceable to the test that drove it, and the agent
starts "improving" code the issue never asked about.

While GREEN, do only **local** tidying of what you just wrote (dead code, a bad name, duplication
inside the slice you touched), running the tests after each step. **Never while RED; get to GREEN
first.**

Anything larger (deepening a module, collapsing a shallow cluster, a cross-file restructure) is a
**separate pass, not this one**. Note it in your handoff notes and route diff-scale restructuring to
`review-diff`, which reads the change against its issue. Resist doing it here even when it is
obviously right.

## Per-cycle checklist

```
[ ] Test describes behavior, not implementation
[ ] Test uses the public interface only
[ ] Test would survive an internal refactor
[ ] Test would survive a legitimate re-tune of a constant
[ ] Code is minimal for this test
[ ] No speculative features added
[ ] No restructuring beyond the slice I just touched
```

**Never hardcode a tuning constant in an assertion.** Derive the expected value from the constant
so a legitimate tune changes numbers, not test outcomes. A test that fails on a tune is measuring
the knob, not the behaviour.

## Mocking

Mock only **true external** dependencies you don't control (payment providers, SMS gateways, third
party APIs). Don't mock your own in-process or local-substitutable collaborators: use the real
thing or a local stand-in (in-memory DB, an embedded Postgres).

<supporting-info>
Rationale, not instruction. The workflow, the checklist and the no-human-gates rule above are what
you follow; nothing here adds a step.

**Why it matters for AI.** TDD is the strongest feedback loop you can give an agent, and
feedback-loop quality is the ceiling on AI output. Writing the failing test _first_ instruments the
behavior before the code exists, so the agent can't write the implementation and then rubber-stamp
it with tests that match the bug. It also accrues genuinely good tests in the repo.

**Philosophy.** Test **behavior through public interfaces**, not implementation details. Code can
change entirely; tests shouldn't. A good test reads like a spec ("user can checkout with valid
cart") and survives refactors. **Bad tests** mock internal collaborators, test private methods, or
verify through a side channel (querying the DB directly instead of the interface); the tell is a
test that breaks on a refactor when behavior didn't change.
</supporting-info>
