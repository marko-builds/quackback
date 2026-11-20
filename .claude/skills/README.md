# Working with Claude Code in this repo

This repo has a shared setup for Claude Code. It is plain files, versioned with the code, so it
belongs to the team and changes the way the code does: by pull request. Using it is the team's
call, change by change; nothing enforces it.

Set up 2026-11-20 as a practice run on a fork of this project; there is no follow-up period.

## What is installed

- `CLAUDE.md` at the root: what the project is, the exact commands CI runs, where things live, and
  the conventions the code already follows. Claude reads it at the start of every session.
- `.claude/rules/*.md`: rules for one area of the code. Each file names the paths it covers and
  loads only when Claude works there.
- `.claude/skills/`: five skills that take a change from idea to reviewed diff, one folder each.
  `<name>/SKILL.md` is the exact instruction Claude follows; read it to see what a step does.
  Licence in `.claude/skills/LICENSE` (MIT).

## The chain

| Step | Skill           | You get                                                                                                                                                                                                                      |
| ---- | --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | `grill-me`      | An interview, in numbered rounds, until you and Claude agree on what to build and what not to. Nothing is written until you confirm.                                                                                         |
| 2    | `write-prd`     | `issues/prd-<slug>.md`, a short PRD (product requirements document): the problem, the decisions, and 3 to 8 checks that define done.                                                                                         |
| 3    | `prd-to-issues` | `issues/NN-<slug>.md`: vertical slices (each a thin piece that works end to end, not one layer), tagged `agent-ready` (a test or build proves it, so Claude can check its own work once someone starts it) or `needs-human`. |
| 4    | `tdd`           | One failing test, then the code that passes it, one behaviour at a time.                                                                                                                                                     |
| 5    | `review-diff`   | Two Claude sub-agents in parallel: one checks the diff against this repo's rules, one checks it against its issue. Plus anything the diff added that nothing calls.                                                          |

Start a step by asking for it ("grill me on this", "write the PRD", "review the diff"). Commit the
`issues/` files with the change they produced: they record why it was built that way.

## Which steps a change needs

- **A new feature, or anything touching several modules:** all five.
- **A bug with a clear cause:** `grill-me` for a few minutes on the fix and its edge cases, then
  `tdd` and `review-diff`. The PRD and issue files are optional.
- **A typo, a copy change, a dependency bump:** none. Just ask.

When in doubt, start with `grill-me`. It costs a few minutes and stops the most expensive mistake:
Claude building the wrong thing well.

## Keeping it useful

- When Claude gets something wrong that a rule would have prevented, add the rule to `CLAUDE.md` or
  to the right file in `.claude/rules/`. Say why in one clause. That is the whole maintenance loop.
- Keep `CLAUDE.md` short. Point at existing docs instead of copying them; a copy goes stale.
- Commands in `CLAUDE.md` should be the ones CI runs. If CI changes, update it in the same PR.
- Personal settings go in `.claude/settings.local.json`. Check that `.gitignore` excludes it while
  still sharing `.claude/rules/` and `.claude/skills/`.

## What this setup does not do

- It does not merge anything on its own. Every change still goes through your normal review.
- It does not replace tests or CI. `review-diff` is a second read of the diff, not a gate.
- It does not run unattended. Each step starts when someone asks for it.
