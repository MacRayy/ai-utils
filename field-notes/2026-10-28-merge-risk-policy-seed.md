# A seed for the risk gate, not just the configs

> After writing `.merge-risk.yml` policies for three repos by hand (a web admin, a React Native app,
> and a .NET backend), the configs were throwaway but the *method* wasn't. The reusable artifact is the
> risk-survey checklist + the gate-wiring traps — so it became a seed.

**Date:** 2026-10-28

---

## Why a seed and not just the tool

`attrecto/merge-risk` (the engine + CLI, incl. the contributed GitLab adapter) is already the reusable
"CLI" — it ships on GitHub and npm. What *wasn't* captured anywhere is how to write a **good policy**
for a given repo. A `.merge-risk.yml` copied from another repo is actively misleading: its zones encode
that repo's risk surface, not yours. Doing it three times surfaced a repeatable method worth seeding:

1. **A stack-aware risk survey.** The same checklist classifies any repo: auth / token-bearing HTTP core
   / secrets / crypto-certs / migrations-or-irreversible-data / money / runtime-config / serving-infra /
   CI → `very-risky`; docs / i18n / tests / stories / assets / mocks → `safe`; everything else → the
   `risky` default. Only the *paths* change per stack; the *concerns* don't.
2. **The three-tier model + why not four.** Tiers are fixed in the engine and, more importantly, are
   valuable because they're unambiguous review contracts. The gradient people want from a "medium" tier
   is better spent as an **approval-count** dial (`very-risky: approvals: 2`), not a fourth label.
3. **The traps.** These cost real debugging during the live rollout and would cost everyone the same:
   - GitLab publishes a commit status **named after the CI job**, so a job named `merge-risk` plus the
     default `merge-risk` status collide → `400 "name has already been taken"`. The gate's
     `MERGE_RISK_STATUS_CONTEXT` must differ from the job name.
   - The CI token must be **masked but NOT protected** — feature-branch MRs run on an unprotected source
     branch, so a Protected variable is invisible and the job silently no-ops.
   - The policy is read from the **base** branch → it must land on the default branch *first* (bootstrap)
     or every MR errors.
   - A GitLab **approval doesn't re-run the pipeline**, so the gate shows a stale "needs approval" until
     the job is re-run (or the webhook listener re-scores it).

## The one judgment call worth surfacing every time

Dependency/lockfile changes: `very-risky` (strict supply-chain) vs `risky` (rely on SCA/audit gates).
Picking `very-risky` makes every routine dependency bump need a deep review — real friction for repos
with automated dep PRs. The seed defaults to `risky` and tells the agent to *ask*, because the right
answer depends on whether the repo already has a dependency-scanning gate.

## Validate, don't trust the globs

Max-tier-across-files + "highest tier wins per file" interact in ways that are easy to get subtly wrong
(a test co-located in a very-risky dir becomes very-risky; a broad `utils is risky` rule can down-tier
the crown-jewel HTTP client). The seed makes validation a required step: run the tool's `evaluate`
against one file per expected tier and confirm the reasons, before wiring CI.
