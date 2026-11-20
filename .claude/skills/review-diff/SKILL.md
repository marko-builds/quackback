---
name: review-diff
description: >-
  Review a diff on two axes in parallel: does it conform to the repo's documented standards, and
  does it faithfully implement the issue it was written for. Use when asked to "review this
  diff/PR/branch", "second pass on these changes", "did this actually implement the issue", or
  before opening a PR on non-trivial work. Reviews changes that already exist.
source-attribution: >-
  Adapted from mattpocock/skills (MIT), engineering/code-review: the two-axis design and smell list.
  Seam check, robustness clause and report format added by this kit.
---

# review-diff

A second pass over a diff, by two reviewers looking for different things. A second pass catches
bugs the author's own reading missed, but only if it has its own angle rather than re-reading the
diff with the same eyes that wrote it. Passing tests is not this: tests show the code does what
the author thought, this checks whether that was the right thing and whether it fits the repo.

**Report-only.** It ranks findings. It never edits code; the engineer driving the session decides
what gets fixed. It is not for an unexecuted plan (nothing to diff), and not for hunting one
specific bug (that is a reproduce, minimise, instrument loop, not a read).

## Input

```
/review-diff                      # working tree vs HEAD
/review-diff <base>..<head>       # an explicit range
/review-diff <PR number>          # via gh, in a repo with a remote
```

Establish three things before reviewing, and say so if any is missing:

1. **The diff.** `git diff <range>`, with the size noted. An oversized diff is a finding in itself.
2. **The standards.** The repo's root `CLAUDE.md` plus any `.claude/rules/*.md` that apply to the
   touched paths, plus `CONTRIBUTING.md` or a coding-standards doc if one exists.
3. **The intent.** The originating issue (`issues/*.md`, or the tracker `CLAUDE.md` names), the PRD,
   or the PR description. **If there is no stated intent, say so and run the standards axis
   alone.** Do not invent a spec and grade against it; that produces confident findings about
   requirements nobody had.

**Re-review.** If the PR already has an earlier review pass, read its comments and the commits
since. Classify each prior finding as Resolved (a fix landed that addresses the specific concern),
Acknowledged (agreed, deferred), Contested (disagreed, record the argument) or No response. Give
both axes this table; only Contested and No-response findings are re-raised.

## The two axes (run as parallel sub-agents)

Each axis gets its own fresh-context sub-agent with the diff, so neither is anchored by the
other's reading. Give each only what its axis needs. Use the strongest model available: an axis is
a verdict, not a scan.

### Axis A: Standards

> Review this diff against the repo's documented conventions. You are given the diff, the root
> `CLAUDE.md`, and the path-scoped rules that apply. Judge only against what those documents
> actually say; a convention you believe in but cannot cite is not a finding. Skip anything
> tooling already enforces.
>
> Then check the diff for these code smells by name and report the ones present, quoting the hunk:
>
> **mysterious name** · **duplicated code** · **feature envy** (a method more interested in another
> class than its own) · **data clumps** (the same few values travelling together) · **primitive
> obsession** · **repeated switches** (the same conditional structure in several places) ·
> **divergent change** (one module changed for many unrelated reasons) · **speculative generality**
> (built for a requirement nobody has) · **message chains** · **middle man** (a class that only
> delegates).
>
> Two more, specific to agent-written code:
>
> - **Narrating comments.** A comment that restates the line below it, or the function's name, is
>   noise. Comment the why, the invariant, the non-obvious edge case. History belongs in git, and a
>   `TODO` pointing at a plan step rots the moment the plan changes.
> - **Enumerated tests.** A long list of near-identical cases is coverage theatre. Ask which three
>   cover the behaviour, and whether any could fail: a test that passes against a gutted
>   implementation is the defect.
>
> A documented repo standard overrides the smell list; where the repo endorses something a smell
> would flag, suppress it. Smells are judgement calls, labelled as such ("possible feature envy").

Naming the smells is the cheapest part of this skill and does most of the work. The labels are
canonical (Fowler, _Refactoring_), so one phrase per smell summons the whole check. Without the
word, a reviewer reviews generically and finds generic things.

### Axis B: Spec

> Review this diff against the issue/PRD it was written to satisfy. For each acceptance criterion
> or stated requirement, decide: **met, partially met, not met, not addressed**, and cite the
> file:line that satisfies it. Then look for:
>
> - **Silent scope drift.** Code that no requirement asked for. Say which.
> - **A safeguard that only exists in a comment.** If a comment claims validation, a guard, or
>   conformance to a spec section, grep for the cited function or section and confirm it exists.
> - **Robustness at the boundaries the requirement does not mention.** For each changed function,
>   name what it does on empty, null, zero, negative and the boundary value, and what a second
>   concurrent caller does to it. Then the regression side: which changed signatures, shared state
>   or removed exports could break a caller that is not in this diff. A requirement is usually
>   written for the happy path; this is the half of "does it work" the spec never states.

This is the axis that catches real bugs. Standards findings are cheap to fix and easy to argue
about; a requirement that quietly went unimplemented is neither.

## How each axis reads

One pass over a diff finds the first thing wrong and stops; the yield is in the passes after that.
Give both sub-agents the same discipline:

> **Loop.** Make a pass, report what you found, then go again over the same hunks until a full pass
> turns up nothing new. Do not stop at the first serious finding.
>
> **Then one explicit second pass**, hunk by hunk, against each named section of your brief in
> turn: the smell list, the agent-code smells, the boundary questions, the rules you were given. A
> pass against "the whole brief" is one you did not really run.
>
> **Report everything you found, including style notes.** Rank and group by severity, but do not
> pre-filter; that judgment belongs to the merged report.

An axis suppresses nothing; the merged report drops what cannot be stated as a failure (below).
Filtering happens once, at the end, where it is visible.

## The seam check (run after both axes, on the merged findings)

Neither axis can see between diffs. A producer and its consumer split across two changes pass both
reviews while connected to nothing. So before reporting, ask the one question the sub-agents
structurally cannot: **what did this diff add that has no non-test caller?** Grep for it. A new
module, export, table, route or flag with no production reference is a finding, not a style note.

## Report

Ranked most-severe first. Every finding needs a file:line and a concrete failure, not a
disposition:

```
# review-diff: <range> (<n> files, <n> lines)

**Intent:** <issue/PRD reviewed against, or "none stated, standards axis only">

## Blocking
- [spec] <one-line claim> (`path/file.ts:120`)
  Requirement "<criterion>" is not met: <what happens instead>.

## Worth fixing
- [standards] <smell name>: <claim> (`path/file.ts:44`)

## Seam
- <what has no non-test caller>, or "nothing added without a caller".

## Noted, not findings
- <things considered and rejected, so the next pass does not re-raise them>
```

**A finding you cannot state as a failure is not a finding.** Drop it. A noisy review trains
people to skim, which costs more than the miss it was trying to prevent.

**Design stance: precision over recall.** A report with a false finding costs more than one that
missed a minor issue, because a false finding teaches readers to skim the real ones. The precision
floor is the loop-then-second-pass discipline above plus the "state it as a failure" filter.

## Chain

`/tdd` (red then green, refactoring deliberately excluded) then **`/review-diff`** (the pass that
owns restructuring findings and spec fidelity), then the engineer fixes what it chooses, then the
PR is opened.
