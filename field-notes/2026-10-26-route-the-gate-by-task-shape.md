# Route the During-Work Gate by Task Shape, and Tier It by Diff Size

> Two token-discipline refinements to the implementation-challenge protocol, learned from running
> the loop on every task for a while: don't run `solution-critic` after *every* change regardless of
> whether a plan existed — route by task shape. And don't review a one-hunk fix on the same model
> tier as a whole feature branch — scale the critic to the diff.

**Date:** 2026-10-26

---

## The problem: one gate for two different jobs

The protocol ran `solution-critic` as the during-work gate after every non-trivial task. But
`solution-critic`'s value is *intent-conformance + completeness* — scope drift, loose ends, missed
wiki updates, reinvented helpers. That pays off when there's a **plan or stated intent to measure
the diff against**. On a small, single-concern change made directly — no plan, no scope to drift
from — that question has almost nothing to bite on, and you still paid for a full critic pass.

The more useful question on an unplanned change is the other one: *are there tactical bugs?* That's
`reviewer`'s job, not `solution-critic`'s.

## What changed: route, then tier

1. **Route by task shape.** Planned / behavior-changing task → `solution-critic` (as before). Small
   *unplanned* non-trivial task → skip `solution-critic` and run `reviewer` instead. Each task is
   gated by exactly one critic during work; the one whose question actually fits the change.
2. **Tier by diff size.** The small-task `reviewer` runs on a **cheaper tier** (`sonnet`, overriding
   its `opus` frontmatter default) — a small diff doesn't need the top tier. A large or
   security-sensitive change still gets the default. The tiering is a per-dispatch override; the
   agent files' `model:` pins are untouched, so nothing about the agents changes.

The thorough whole-branch `reviewer` still runs once at PR time on its default (`opus`) tier — this
only changes the *during-work* gate, not the PR-time review.

## Why this fits the existing design

The [one-knobs-file field note](2026-10-21-checks-file-and-verify-gate.md) added rtk to make
"critic loop on every task" affordable by compressing the bash output. This is the same goal from
the other side: instead of making each pass cheaper, run the *right* pass and size it to the change.
The critics still default to `needs_revision` and still gate the next step — they just stop spending
`solution-critic`'s intent-conformance budget on changes that never had an intent to conform to.
