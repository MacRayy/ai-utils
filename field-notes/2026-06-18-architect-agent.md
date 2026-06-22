# Teaching Claude to Step Back: The Architect Agent

> A new critic agent that scouts alternative solutions before we draft a plan — and is
> deliberately blindfolded to the source code so it doesn't just suggest "do more of what's
> already there." Tradeoffs it accepts get auto-captured as `wiki/tech-debt/` entries.

**Date:** 2026-06-18
**Updated:** 2026-06-19 — added "Where else could it run?" after exploring two extension ideas

---

## The problem with our existing critics

We already have three critic agents: `plan-critic`, `reviewer`, and `solution-critic`. They
catch real bugs and have saved us multiple times. But they share a blind spot — they all run
on a *chosen* solution and ask "is this implementation correct?" None of them ask "is this
the right shape of solution in the first place?"

When the existing reviewer reads a diff, its instinct is "look how the rest of the codebase
solves similar problems and check whether this fits." That's the right instinct for tactical
review. It's the wrong instinct for strategic review, because it bakes in the existing
patterns as the answer. If everything in the codebase uses pattern X and someone proposes
pattern Y for a new problem, the reviewer says "Y is inconsistent, use X." That's only
correct if X is actually the right pattern. If X is the residue of an old constraint that no
longer applies, the reviewer protects the wrong thing.

We wanted an agent that could step back, look at the problem fresh, and propose alternatives
without anchoring on what we'd already built.

## What we designed

A new agent named `architect`. Three traits make it interesting:

**1. It runs before we draft the plan, not after.**
The existing `plan-critic` runs *after* a plan is drafted, to challenge whether the plan is
well-scoped. The architect runs *before* drafting, when the question is still "what shape of
solution are we even reaching for?" Putting strategic review after the plan is sunk-cost — by
then we've invested in a direction and the critic mostly catches scope drift. Strategic
review needs to land before the investment.

**2. It cannot read the source code.**
This is the part that surprised people on the team. The architect can read `wiki/**/*.md`
and `**/*.test.*` and `modules/e2e/features/**`, but it's prompted to never read `*.ts` /
`*.tsx` outside test suites. It must reason from the wiki (architecture, decisions, glossary,
features, tech-debt, hacks) and from tests (the behavioral contract), but not from the
implementation.

We did this deliberately to break the "look how it's done now → recommend more of the same"
loop. If you give a reasoning model the existing solution and ask "what should we do?", it
will pattern-match. Hide the existing solution and force it to reason from principles, and it
starts proposing genuinely different shapes. Half the time the existing pattern is still the
right call — but now we know that because we considered the alternatives, not because the
agent was anchored.

A secondary effect we didn't fully expect: this forcing function is great for the wiki. If
the wiki is too thin for the architect to reason from, it says so explicitly ("wiki gaps
surfaced") instead of falling back to the source. That gives us a steady pressure to keep the
wiki current — exactly the dynamic Karpathy describes with LLM wikis as load-bearing infra.

**3. The recommendation writes itself into the wiki.**
A recommendation surfaces facts about the system that often belong in the wiki, not just in
the architect's response — and we want them captured at the moment of decision, before
anyone forgets. So the architect has `Write` and `Edit` access across `wiki/` and a checklist
of which categories to populate when:

- The recommendation itself, if it's a non-obvious choice → `wiki/decisions/<date>-<slug>.md`
  with `Status: proposed`. The main agent flips it to `accepted` once the plan is approved.
- Accepted compromises → `wiki/tech-debt/<slug>.md`.
- Workarounds with a concrete "remove when X" condition → `wiki/hacks/<slug>.md`.
- New domain terms → `wiki/glossary.md`.
- New external dependencies → `wiki/integrations/<slug>.md`.
- Architectural shifts → `wiki/architecture.md`.

Plus a `wiki/_log.md` line for every page touched, per the schema. The architect is
explicitly *not* allowed to write to `bugs/`, `runbooks/`, `features/`, or `workflows/` —
those are post-fact categories (recorded after a bug is fixed, after an incident, after a
feature ships) and don't make sense at recommendation time.

Most recommendations produce zero or one wiki page. Significant ones might produce two or
three. The architect is prompted to skip categories that don't apply rather than manufacture
content to fill the checklist.

This solves a real problem: tradeoffs and decisions we make tend to evaporate from memory
within a quarter. Six months later someone hits the friction and asks "why did we do it this
way?" and nobody remembers. The architect's wiki entries are the answer to that question,
written before anyone forgets.

(Initially we only allowed writes to `wiki/tech-debt/`. Within a day of shipping we expanded
to the categories above. The reasoning the architect generates often belongs in `decisions/`
or `hacks/` just as much as in `tech-debt/`, and restricting writes to one folder was
arbitrary.)

## How it integrates

The plan mode protocol in `CLAUDE.md` now runs the two critics in sequence:

```
1. Architect (NEW) — scouts alternatives from the problem statement (no plan yet)
   → returns ranked candidates + recommended path + tech-debt entry for the
     compromises baked into the recommendation
2. Main agent drafts the plan using the architect's recommendation, or overrides
   with a recorded rationale
3. Plan-critic — challenges the drafted plan against the actual codebase
4. Loop plan-critic until approved
5. ExitPlanMode
```

Two agents, two jobs. Architect asks "is this the right shape of solution?" Plan-critic asks
"given that shape, is the plan well-scoped and grounded?" Reversing them wastes the architect's
challenge on a sunk-cost draft.

## Where else could it run?

If it's useful before drafting, would it also be useful after implementation or before
opening a PR? Two extensions, two outcomes — and the same property kept deciding them.

**1. After implementation, before declaring the task done — skipped.**
The natural next question is "did we build the thing we said we'd build?" but that's exactly
`solution-critic`'s job, and that agent gets to read the diff. Architect, still blindfolded,
would be guessing at what shipped from wiki and tests alone. Weaker signal, same answer. The
one narrow exception is *drift detection*: re-invoke architect with its own original
recommendation in hand, and ask "did we actually build candidate A, or did we slide into B or
C under implementation pressure?" That's a question solution-critic can't answer because it
has no baseline. Kept as an ad-hoc tool, not a protocol step.

**2. Before opening a PR — kept, with a narrow trigger.**
Added to `CLAUDE.md` as an optional pre-PR self-check. The trigger is intentionally tight:
new pattern, new module, new cross-module dependency, new top-level feature surface. For
those PRs, dispatch architect against the *PR description* (not the diff) and ask "given
this is what we built — knowing only the contracts and wiki — would you still recommend this
shape?" The blindfold turns into a feature here: if a fresh-eyes architect would arrive at
the same shape from the wiki alone, the shape is defensible and the wiki is doing its job.
If not, either we drifted or the wiki is now lying about the system — and that's worth
reconciling before human reviewers spend cycles on it. Typical bugfix and feature-extension
PRs skip the check; cost-to-value is bad when the shape isn't in question.

The throughline: the blindfold isn't a quirk of the plan-mode use case. It's the property
that makes architect's output trustworthy. Each candidate extension lives or dies by whether
the blindfold survives — preserve it and the agent stays useful, break it and the output
collapses into either rubber-stamping or manufactured alternatives. Worth remembering when
the next "what if it could also..." question comes up.

## First field test

First spin on a real production bug — a debugging-investigation task that arguably wasn't
the architect's home turf. From wiki + tests + schema alone, the architect correctly
identified the root cause (a forgotten TODO stub in a DTO mapper), and went further:
flagged this as the third recurrence of the same "frontend stubs a value while waiting on
backend, then never removes the stub" pattern in two months, cited the prior decisions, and
proposed two structural countermeasures. The blindfold leaked in one place — it ran `grep`
against source and read the matched lines, which the prompt's "search for existence, don't
read the body" rule technically forbids. Tightening the prompt would close the gap; the
result here didn't suffer.

The two critics earned their slots. **Plan-critic** ran four rounds, each adding a downstream
consumer the architect couldn't see (because the architect couldn't follow dataflow into
source and the wiki didn't enumerate every callsite). Without it the fix would have shipped
a symptom-cure + new regression at a different surface. **Solution-critic** found three more
issues post-implementation: a test case that wasn't independently falsifiable, a truth-table
corner the plan had punted that turned out to matter, and a minor wiki-schema deviation. Two
were real correctness improvements; one was honestly deferred.

The total round count (4 + 2) was higher than I'd have guessed for what started as a two-line
bug fix, and the final diff covered four files plus a postmortem. But each round genuinely
added something the previous stage had missed. The loop was arguably *more* valuable on a
small fix than it would have been on a big architectural decision — small fixes are the
easiest to ship half-done.

## What we're watching for

A few open questions we'll learn from over the next month:

- **Does the no-source-access rule actually break anchoring?** Or does the architect end up
  proposing reasonable-sounding alternatives that are actually impossible given constraints
  in the source code it never saw? If alternatives consistently die in implementation, we'll
  need to widen its read scope (or improve the wiki).
- **Does the tech-debt entry get written for every recommendation, or only when there's real
  debt?** We're trusting the agent's judgment that "not every choice creates debt." If we end
  up with `wiki/tech-debt/` polluted with non-debt entries, we'll tighten the prompt.
- **Is opus the right model?** We chose opus because the architect's job is reasoning-heavy:
  generating genuinely distinct alternatives, identifying cross-cutting tradeoffs, catching
  when the problem framing is wrong. Sonnet might be fine; we'll see whether opus's edge
  earns the extra latency on the kinds of problems we actually throw at it.

## Why this fits with the other critics

The three existing critics — `plan-critic`, `reviewer`, `solution-critic` — each catch a
different class of mistake, and the architect adds a fourth. It's worth being explicit about
how they divide the work:

| Agent | When it runs | What it asks |
|---|---|---|
| **architect** | Start of plan mode, and optional pre-PR check on novel work | Is this the right shape of solution? Would fresh eyes still recommend it? |
| **plan-critic** | After plan draft, before ExitPlanMode | Is the plan well-scoped, grounded in the codebase, free of reinvented utilities? |
| **reviewer** | Before commit, on PR-style review requests | Are there tactical bugs in the code? Prop-rest leaks, key collisions, timezone bugs? |
| **solution-critic** | After tests pass, before declaring done | Is there scope drift? Loose ends? Missed wiki updates? |

The architect is the only one of the four that doesn't get to see the code. That's a
feature, not a bug.

