# Merge-risk Policy Seed

A single-file seed that bootstraps a **risk-based merge-review gate** for a repo: it generates a
`.merge-risk.yml` tuned to the repo's real risk surface, and wires the CI job (GitHub or GitLab) that
scores each pull/merge request and gates it on the review that risk calls for. Drop into any repo, hand
to a coding agent, get a working tier-gate on the next PR.

## What this is

[`attrecto/merge-risk`](https://github.com/attrecto/merge-risk) is a deterministic, rule-based scorer:
it assigns every PR/MR a tier — `safe` / `risky` / `very-risky` — and the tier decides the review. The
tool ships the engine; the **policy** (`.merge-risk.yml`) is what you tune per repo. This seed is the
method for writing a *good* policy (not a generic copy) plus the CI wiring and the gotchas.

- **Deterministic, not AI.** The tier is computed from path globs, content patterns, and change size —
  reproducible and un-gameable. Risk is the **max** across changed files, never averaged, so one
  dangerous file can't hide among a hundred safe ones.
- **Read from the base commit.** The gate reads the policy from the PR's *target* branch, so a PR can't
  loosen the rules that judge it.
- **The output.** A `merge-risk:<tier>` label, a report comment explaining the tier, and a commit
  status; the required check / job exit code is the enforcement.

The seed's value is the **risk-survey method** (§B) — the same checklist used to classify a web app, a
mobile app, and a backend consistently — plus the **gate wiring** (§E) and the **traps** that bite
everyone the first time (especially on GitLab).

## How to use this file (for humans)

1. Copy `merge-risk-seed.md` into the root of the target repo (or just point an agent at this URL).
2. In a fresh agent session, paste:

   > Read `merge-risk-seed.md` and follow it: survey this repo's risk surface, then show me the
   > `.merge-risk.yml` you propose (with per-zone justifications and the judgment calls) before writing
   > anything. Don't wire the CI job until I approve the policy.

3. Review the proposed policy and answer the judgment calls (§F) — mainly: is a dependency/lockfile
   change `risky` or `very-risky`, and how many approvals each tier needs.
4. Approve; the agent writes `.merge-risk.yml`, validates it, and adds the CI job.
5. **Bootstrap:** merge the policy to the default branch first (the gate reads it from the base), then
   open a PR to see it score.

## How to use this file (for the LLM agent reading it)

When the user asks you to bootstrap a merge-risk policy from this seed:

1. **Survey the repo's risk surface** (§B). Walk the tree; identify the security/access/irreversible/
   money/infra files and dirs. For a big repo, dispatch a read-only exploration sub-agent with the §B
   checklist and keep only its path map — don't dump the tree into your context.
2. **Draft `.merge-risk.yml`** (§C): group the surveyed paths into the three tiers, write zone globs
   with a one-line justification comment each, set the size bump + excludes, and write the per-tier
   reviewer guides. Prefer dir/glob patterns over listing every file.
3. **Show the user the draft and the judgment calls (§F) before writing it.** Do not wire CI yet.
4. **Validate** (§D): run the tool's `evaluate` against a few representative changed files and confirm
   the max-tier + per-file reasons match intent. Fix the zones if a file lands in the wrong tier.
5. **Wire the gate** (§E) only after the policy is approved — GitHub Action or GitLab CI job, with the
   traps handled. Then tell the user the bootstrap step (policy must reach the base branch first).
6. **Do not delete `merge-risk-seed.md`** unless asked. Leaving it documents the bootstrap.

---

## §A — The tier model

Three tiers, each a **clear, actionable review contract**:

| Tier | Approvals (default) | What the reviewer does |
|---|---|---|
| `safe` | 0 | Nothing. Merges once checks are green. |
| `risky` | 1 | Reviews the *shape* — architecture, interfaces, reuse — and tries it on a deployed env. |
| `very-risky` | 1 (consider 2) | Reviews *in depth*, implementation included; checks the rollback path. |

**Why three, not four.** The tier count is fixed in the engine (`safe`/`risky`/`very-risky`) — adding a
"medium" would mean forking the tool, and it blurs the one thing tiers are good at: being an unambiguous
instruction. A clean ladder is *nothing / review-the-shape / review-in-depth*. The gradient you'd want
from a 4th tier is better expressed with the two existing knobs:

- **Approval count per tier** — e.g. `very-risky: approvals: 2` makes the dangerous stuff need two
  humans while ordinary `risky` needs one. This is the real "how much weight" dial.
- **The per-tier `guide`** text already differentiates review depth.

Reserve a 4th tier only for a team that genuinely runs four distinct review *protocols*.

---

## §B — The risk-survey method (how to classify)

Classify every path into one of three buckets. **`risky` is the default** (`default-tier: risky`), so
you only write zones for `safe` and `very-risky`; everything unlisted is `risky`. Highest tier wins when
a file matches more than one zone.

### VERY-RISKY — security / access / irreversible / money / infra

Look for these, by concern (names vary by stack — map to the repo's reality):

- **AuthN / AuthZ** — login, token issuance/validation, session, SSO/OAuth/OIDC/JWT/Keycloak, RBAC /
  permission predicates (`hasPermission`, policy handlers), route/endpoint guards, access-escalation
  (role changes), token-bearing invites.
- **The token-bearing HTTP core** — the one HTTP client/interceptor that injects the auth token and does
  refresh-on-401. (This single file is usually the crown jewel; make sure a broad "utils is risky" rule
  doesn't down-tier it.)
- **Secure storage & secrets** — secure token storage (keychain / secure-store), and secret-bearing
  files: `.env*` (real ones, not `*.example`), `appsettings*.json`, `secrets/`, keystores, service-
  account creds, API-key config.
- **Crypto / certificates / TLS** — certificate pinning (SPKI pin lists), mTLS cert validation, crypto
  key handling, signing, networking-security config. Pin lists are rotation time-bombs — always gate.
- **Irreversible data** — DB schema/data **migrations**, deletion / anonymization (GDPR), destructive
  admin actions, seed/`.sql`.
- **Money** — payments, billing, credits/points/**ledger**, subscriptions / in-app purchase, loyalty,
  coupons, order fulfillment/pricing, manual credit grants.
- **Runtime / env config & routing** — the config that sets the backend URL / feature flags / keys; the
  route tree + guards.
- **Serving / build / infra** — reverse proxy (nginx: security headers, CSP), Dockerfile, runtime-env
  injection scripts, **native/mobile build** (`android/`, `ios/`, Gradle, `Info.plist`, Firebase config
  files), **the CI pipeline file itself** (it deploys *and* runs the gate → always very-risky), and the
  composition root / DI / middleware pipeline.
- **Real-world-action workers** — background jobs that send notifications, trigger device commands,
  grant payouts, or run replication.

### SAFE — no runtime behavior

- Docs / markdown; i18n / translations / locales; **tests** (all test globs — watch for repos with a
  **dual harness**, e.g. `*.test.ts` *and* `*.test.tsx`); e2e suites; Storybook stories + config; static
  assets; mock servers / mock data; planning docs (stories / prd / tasks).

### RISKY — the default (write no zone)

Everything else: ordinary feature/UI/domain code, non-sensitive services, non-runtime build config, and
— by common decision — dependency/lockfiles (see §F).

### Stack hints

- **Web (React/Vue/…):** very-risky = `src/**/auth*`, the axios/fetch client + interceptors, route
  config/guards, `src/config/**` (env), nginx/Dockerfile/entrypoint, `.env*`, `.gitlab-ci.yml`/
  `.github/workflows/**`. Safe = `public/locales/**` or `src/locales/**`, `**/*.test.*`, `e2e/**`,
  `**/*.stories.*`, `src/assets/**`.
- **Mobile (React Native / Expo):** add cert-pinning plugins/scripts, `src/utils/secureStore*`, the
  native dirs `android/**` + `ios/**`, Firebase config files (`google-services.json`,
  `GoogleService-Info.plist`), `app.config.*`/`eas.json`, `.secrets/**` (signing keys). Note the root
  `config/` is often *test* (Appium/Detox) config → safe, distinct from `src/config/` (runtime) →
  very-risky.
- **Backend (.NET / Java / Node):** very-risky = `**/Migrations/**` + `**/*DbContext*`/ORM schema,
  `**/Auth/**` + authorization handlers, mTLS/cert handling, `**/appsettings*.json` + `secrets/`,
  rate-limiting, audit logging, the money feature folders, `**/Program.cs`/startup/DI, the privileged
  admin surface, and workers with side effects. Safe = `**/*.Tests/**`, `**/*.http`/Postman, `docs/**`.

---

## §C — Writing `.merge-risk.yml`

Start from this skeleton and fill the zones from your survey. Keep each glob commented with *why*.

```yaml
# How risky a change is to merge, and the review that risk calls for. Scored by attrecto/merge-risk;
# https://github.com/attrecto/merge-risk#policy. Judged by the copy on the PR's base branch.
version: 1

default-tier: risky

tiers:
  safe:
    approvals: 0
  risky:
    approvals: 1
    guide: >-
      Review the shape of the change — architecture, interfaces, reuse — and try it on a deployed env.
  very-risky:
    approvals: 1 # consider 2 — two humans for the dangerous stuff
    guide: >-
      Review in depth, implementation included. Auth, secrets, migrations, money and infra are hard to
      undo once live — check those and the rollback path.

zones:
  - tier: safe
    paths:
      - '**/*.md'
      - '<i18n/locales path>/**'
      - '**/*.test.*' # include every test glob the repo uses (dual harnesses: *.test.ts AND *.test.tsx)
      - '<e2e dir>/**'
      - '**/*.stories.*'
      - '<assets dir>/**'

  - tier: very-risky # highest tier wins; these override the default for the files they cover
    paths:
      - '<auth / token / access-control paths>' # …one line of justification each
      - '<the token-bearing HTTP client>'
      - '<secrets: .env, appsettings*.json, secrets/**, keystores>'
      - '<crypto / cert-pinning / mTLS>'
      - '<migrations / deletion / irreversible data>'
      - '<money: payments / billing / ledger / loyalty / orders>'
      - '<runtime/env config + routing>'
      - '<serving/build/infra: nginx, Dockerfile, native build, the CI file>'

size:
  bump-above-lines: 600 # a large diff is bumped one tier
  exclude: # generated / non-runtime files that shouldn't cost a PR its real tier
    - '<lockfile>'
    - '**/*.snap'
    - '<i18n>/**'
    - '**/*.test.*'
    - '<migrations>/**' # large generated migrations shouldn't bump an unrelated tier (still very-risky by zone)
```

Guidance:

- **Prefer dir globs** (`src/features/auth/**`) over listing files, but call out the few individual
  crown-jewel files (the HTTP client, the pin list) explicitly so a broad default can't down-tier them.
- A whole-dir very-risky zone **over-flags** a few benign files in it (and any co-located test becomes
  very-risky, since max-tier wins). That's the safe direction — accept it, or scope the glob tighter.
- `content patterns` (regex on added lines, e.g. SQL `DROP|TRUNCATE`) are available too — see the tool
  README. Skip them in v1 unless there's an obvious one; they're false-positive-prone.

---

## §D — Validate before shipping

Run the tool against a few representative changed files and confirm the tier + per-file reasons:

```sh
# In a throwaway git repo (or the real one): commit the policy on `main`, make a change on a branch
# touching one file per expected tier, then:
npx @attrectodev/merge-risk evaluate --base main
```

Confirm: a crown-jewel file → `very-risky`, an ordinary feature file → `risky`, a doc/locale/test →
`safe`, and the MR max-tier is the highest among them. If a file lands in the wrong tier, fix the zone.

---

## §E — Wire the gate

### GitHub

```yaml
# .github/workflows/merge-risk.yml
name: merge-risk
on:
  pull_request_target: { types: [opened, synchronize, reopened, edited, ready_for_review] }
  pull_request_review: { types: [submitted, edited, dismissed] }
permissions: { contents: read, pull-requests: write, issues: write, statuses: write }
jobs:
  gate:
    runs-on: ubuntu-latest
    steps: [{ uses: attrecto/merge-risk@v0 }]
```

Then require the `merge-risk` status in a branch ruleset. GitHub re-runs on `pull_request_review`, so an
approval flips the status automatically.

### GitLab

```yaml
# in .gitlab-ci.yml
merge-risk:
  image: node:22-alpine
  stage: <an existing stage, e.g. lint or code-quality>
  needs: []
  interruptible: true
  allow_failure: true # advisory while piloting; drop it to make it a hard gate
  cache: []
  variables:
    # MUST differ from this job's name — see trap #1
    MERGE_RISK_STATUS_CONTEXT: merge-risk-verdict
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
  script:
    - npx -y @attrectodev/merge-risk@v0 gitlab-gate
```

Set a masked CI variable `MERGE_RISK_TOKEN` = a project/group access token (Developer role, `api`
scope). Make the job a gate via **Settings → Merge requests → "Pipelines must succeed"**.

### Traps (GitLab especially)

1. **The status name must differ from the CI job name.** GitLab publishes a commit status named after
   the job, so a job named `merge-risk` + the default `merge-risk` status collide → `400 "name has
   already been taken"`. Set `MERGE_RISK_STATUS_CONTEXT` distinctly (or rename the job).
2. **`MERGE_RISK_TOKEN` must be masked but NOT protected.** Feature-branch MRs run on an unprotected
   source branch; a Protected variable is invisible there and the job silently fails to authenticate.
3. **The policy is read from the base branch → bootstrap.** Land `.merge-risk.yml` on the default branch
   first; until then every MR errors `No .merge-risk.yml on the base branch`.
4. **An approval doesn't re-run the pipeline (GitLab).** After approving, re-run the job (it reads
   approvals live) — or run the `gitlab-webhook` listener, which re-scores on approval automatically.
5. **Reading `/projects/:id/approvals` needs Maintainer.** With a Developer token it 403s and the gate
   assumes reset-on-push is off (a per-run warning). Enabling **"Remove all approvals when commits are
   added"** is a setup step, not something the gate reads.
6. The gate counts an approval only from **someone other than the author** — a self-approval never
   satisfies the tier.

---

## §F — Judgment calls to confirm with the user

- **Dependency / lockfile / build-config tier.** `very-risky` (strict supply-chain) blocks every routine
  dep bump behind a deep review; `risky` relies on your SCA/audit/lint gates instead. **Default to
  `risky`** and let the audit gates own supply-chain — but ask, especially if there's no SCA gate.
- **Approval counts.** Keep all at 1, or raise `very-risky` to 2 for a two-person review of
  auth/secrets/money/migrations.
- **Whole-dir vs file-scoped very-risky** for config dirs that mix sensitive and benign files (whole dir
  = simpler + safe-direction; file-scoped = less noise).
- **Operational-but-reversible services** (order status, reward lifecycle, event triggers): usually
  `risky`, but elevate if a wrong change has real-world blast radius (e.g. load-shedding, payouts).

---

## Customization seams

- **Tool source.** This seed references `attrecto/merge-risk` and `@attrectodev/merge-risk`. GitLab
  support (`gitlab-gate` / `gitlab-webhook`) is a contributed adapter; if it isn't in the published
  package yet, run the self-contained bundle from a pinned fork commit
  (`node <(wget -qO- raw.githubusercontent.com/<fork>/<sha>/dist/cli/index.mjs) gitlab-gate`) until a
  release ships, then switch to `npx @attrectodev/merge-risk@v0 gitlab-gate`.
- **i18n / e2e / assets paths** differ per repo — fill the `<…>` placeholders from the survey.
- **Reviewer guides** — rewrite the `guide` text in the host's voice / language.
- Pairs well with `agents-seed.md`: the merge-risk gate is the deterministic companion to the LLM
  critic loop — one scores *where to look*, the other *looks*.
