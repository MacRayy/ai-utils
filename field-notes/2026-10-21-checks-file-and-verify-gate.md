# One Knobs File, a Verify Gate, and a Loop That Teaches Itself

> Three changes to the agents seed, all learned from running the critic loop on a real
> multi-repo product for a few months: move every project-specific knob out of the agent files
> into one `.claude/checks.md`; make `solution-critic` refuse "tests pass" as proof a
> user-visible change works; and add a `trap-learn` agent that promotes a recurring finding into
> a durable check.

**Date:** 2026-10-21

---

## The problem with baking knobs into agents

The first version of `agents-seed.md` substituted project-specific values — helper locations,
traps, reviewer patterns, forbidden files, the default branch, the four gate commands — directly
into each agent file at bootstrap time. It worked, but three frictions showed up once the loop
was running across several repos day to day:

1. **The same knowledge lived in four files.** `helper_locations` was pasted into both
   `plan-critic` and `solution-critic`; `default_branch` into four agents. Tighten a trap in one
   and the others drifted. There was no single place that answered "what does this project consider
   a footgun?"
2. **The agents weren't reusable.** Because each file had a repo's specifics compiled in, you
   couldn't share one set of agent files across repos — every repo got its own mutated copies. For
   a team maintaining many repos, that's N copies to update whenever an agent's prompt improves.
3. **Nothing could extend them cleanly.** We wanted the loop to *learn* — to turn a finding the
   critics raised twice into a permanent check. Appending a rule meant editing an agent file and
   picking which of the duplicated copies to touch. Fragile.

## What we changed: `.claude/checks.md`

All project-specific knobs now live in one file — `default_branch`, `commands`,
`helper_locations`, `forbidden_files`, `project_traps`, `conventions`, `reviewer_patterns`,
`test_globs` — each a `## <section>` heading. The agent files are now **generic**: they read the
relevant section at runtime and fail soft (fall back to generic judgment, and say so) if the file
or a section is missing.

Three things fell out of that one move:

- **One source of truth.** Edit a trap once; every critic sees it.
- **The agents became shareable.** With nothing project-specific compiled in, the same agent files
  work in any repo — so a team can ship them as a Claude Code **plugin** and keep only
  `.claude/checks.md` per-repo. Agent improvements ship once instead of being re-copied. (We added a
  "Packaging for a team" note to the seed, including the rule that installing a plugin must add
  capabilities, never silently start a background job on a teammate's machine — keep schedulers
  local to whoever operates them.)
- **The loop can now teach itself** — which is the next change.

## `trap-learn`: findings → durable checks

A critic catching the same class of bug every quarter is a smell: the lesson never became a rule.
`trap-learn` is a small agent whose only job is to take a concrete finding — a trap, a reinvented
helper, a convention violation, a reviewer-class bug — and file it into the right section of
`.claude/checks.md`, de-duped and phrased as something grep-able ("`new Date('YYYY-MM-DD')` parses
as UTC midnight — use `parseLocalDate`"). The implementation protocol dispatches it after
`solution-critic` approves, when a finding has shown up more than once.

The payoff compounds: one finding, filed once, is caught automatically by `solution-critic` on
every future task. The externalized knobs file is what makes this clean — there's exactly one place
to append to.

## The verify gate: "tests pass" is not "it works"

The sharpest behavioral lesson. Green unit tests routinely pass on a user-visible change that is
visibly broken, because the mocks were written to match the implementation rather than reality.
`solution-critic` now treats a user-visible change with **no behavioral evidence** — a passing
end-to-end spec that drives the real flow, or a recorded manual verification — as `needs_revision`.
The implementation protocol adds a verify step *before* the critic runs, so the evidence exists by
the time the critic looks for it. Pure refactors with unchanged behavior are exempt; anything a user
can see is not.

## A note on cost, and on honor-system protocols

Two smaller things worth recording:

- **Running several critics per task is bash-heavy**, and most of the `git diff` / lint / test
  output is noise the model pays to read. We added a pointer to [rtk](https://github.com/rtk-ai/rtk),
  a command-output compressor that cuts 60–90% of those tokens on the bash tool call without
  touching any agent. It's the cheapest way to make "critic loop on every task" affordable.
- **The §B protocols are still honor-system** — "only call `ExitPlanMode` once the critic
  approved." In a project where that mattered we backed it with a `PreToolUse` hook that actually
  blocks the tool until the verdict is present. We left the hook out of the generic seed on purpose
  (it's an operator-level enforcement choice, fail-open and workspace-scoped, not something to
  impose by default), but it's the obvious next step if a team finds the protocol getting skipped.

## Why this fits the existing design

The [architect field note](2026-06-18-architect-agent.md) argued the critics work because each
asks a different question the others can't. These three changes don't add a new question — they make
the existing critics *sharper and cheaper to keep sharp*: `solution-critic` now asks "is the
behavior actually verified?" on top of its old questions; `trap-learn` turns its findings into
standing checks; and `.claude/checks.md` is the shared memory that keeps every critic tuned to this
specific codebase from one editable file.
