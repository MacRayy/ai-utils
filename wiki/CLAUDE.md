# Wiki schema

This folder is the project's LLM-maintained knowledge base. It captures everything the code alone
doesn't show: architectural patterns, decisions and their rationale, bug postmortems, known hacks
and tech debt, integration contracts, runbooks, in-flight projects, and the domain glossary.

## Layers (strictly separated)

- **Sources** — read-only ground truth: the codebase, `git log`, GitHub/GitLab PRs, agent
  transcripts, ad-hoc notes a human drops in.
- **The wiki** — this folder. Markdown pages the LLM creates and updates. Sources stay immutable;
  the wiki is a derived artifact that the LLM keeps current as code changes.
- **The schema** — this file. Defines *when* to update the wiki, *how* to write pages, what
  categories exist, and how the taxonomy evolves.

## When to update (treat as a checklist for every non-trivial task)

Before finishing a task, walk this list. If a row applies, write or update the matching page in
the same commit as the code change.

| Trigger | Where to write |
|---|---|
| Architectural shift, new layer, new cross-module pattern | `architecture.md` + maybe a `decisions/` page |
| Non-obvious decision (vendor switch, flag flip, schema split, deprecation) | `decisions/YYYY-MM-DD-slug.md` |
| Non-obvious bug fix with a real root cause | `bugs/YYYY-MM-DD-slug.md` |
| Workaround landing with a "remove when X" condition | `hacks/slug.md` |
| Accepted longer-lived debt | `tech-debt/slug.md` |
| New external system integrated, or an existing one changing contract | `integrations/slug.md` |
| Recurring operational pain point with a known recovery path | `runbooks/slug.md` |
| User-visible capability added or significantly changed | `features/slug.md` |
| New domain term used in code or PR descriptions | `glossary.md` |
| New repeatable team process (PR flow, release dance, codegen loop) | `workflows/slug.md` |

After writing or updating any page, append a one-line entry to `_log.md` in chronological order:
`- YYYY-MM-DD — title — one-sentence summary — commit-or-PR`

## Page templates

Keep pages tight. Every page has a one-line summary at the top so the index can quote it.

### `decisions/YYYY-MM-DD-slug.md`

```markdown
# <title>

> One-line summary.

**Date:** YYYY-MM-DD  **Status:** proposed | accepted | superseded by `[[link]]`

## Context
Why this came up. What constraint forced a choice.

## Options considered
- A — pros / cons
- B — pros / cons

## Decision
What we picked, in one short paragraph.

## Consequences
What changes downstream. What we're now committed to. What gets harder.

## References
- `[[decisions/...]]` superseded
- PR #N, commit `abc1234`
```

### `bugs/YYYY-MM-DD-slug.md`

```markdown
# <title>

> One-line summary.

**Symptom** — what the user or monitoring saw.
**Root cause** — what was actually wrong, in plain language.
**Fix** — what changed, with file refs.
**Lessons** — what to look for next time. Link any `runbooks/` page that came out of it.

## References
PR #N, commit `abc1234`
```

### `hacks/slug.md`

```markdown
# <title>

> One-line summary.

**What it does** — short, neutral description.
**Why it exists** — the constraint that forced the workaround.
**Remove when** — the concrete condition that retires the hack. If you can't state one, this is
debt, not a hack — move it to `tech-debt/`.
**Files** — paths involved.
```

### `tech-debt/slug.md`

```markdown
# <title>

> One-line summary.

**What it is** — what's not ideal.
**Why it's that way** — original constraint.
**Current cost** — concrete pain it causes today.
**When to fix** — trigger that would justify the work. Often "next time we touch X".
```

### `runbooks/slug.md`

```markdown
# <title>

> One-line summary of when to use this runbook.

## Trigger
What you saw that made you open this page.

## Steps
1. Concrete command or click.
2. ...

## Verification
How you know it worked.

## Related
Link any `bugs/` postmortem that inspired this runbook.
```

### `features/slug.md`

```markdown
# <title>

> One-line summary of the user-visible capability.

**Where** — route(s) or entry point in the app.
**What users see** — the screen, the controls, the data.
**How it's wired** — top-down: page → components → hooks → API call → backend endpoint.
**Specs** — paths to E2E specs or other behavioural tests covering this feature.
```

### `integrations/slug.md`

```markdown
# <title>

> One-line summary of the external system.

**What we use it for** — surfaces in the app that depend on it.
**Contract** — the shape of the request/response we rely on. Cite OpenAPI / vendor doc URL.
**Auth & config** — where credentials live, which env vars matter.
**Gotchas** — quirks we've hit. Link `bugs/` postmortems.
```

### `apps/slug.md`

One per runtime surface (each module, service, or app that has its own dev/build/run lifecycle).

```markdown
# <name>

> One-line summary.

**Purpose** — why this surface exists.
**How to run it** — the commands.
**Layers** — top-down dependency on other modules.
**Notable internals** — anything non-obvious to a new reader.
```

### `workflows/slug.md`

```markdown
# <title>

> One-line summary.

**When** — the trigger.
**Steps** — numbered, with exact commands.
**Gotchas** — anything you have to remember.
```

## Tone & length

- One-line summary at the top of every page. The index re-uses it.
- Bullet points over prose. Code refs as inline backticks (`file_path:line`).
- Cross-link with relative paths in `[[...]]` form for graph view, plus markdown `[label](path)`
  for tools that don't grok wikilinks.
- If a page grows past ~200 lines, split it. If a folder gets >20 files, group by sub-folder.

## Evolving the schema

Categories exist to make the page easy to find. When content stops fitting:

1. Propose a new folder or page-template here in `CLAUDE.md`.
2. Add it to the directory tree in the project README.
3. Add a `_log.md` entry explaining why.

When in doubt, don't write. A noisy wiki is worse than a quiet one. Skip mechanical fixes,
formatting churn, type bumps, dependency upgrades — unless they encode a real decision.

## Exclusions

The wiki is markdown content the agent maintains. Treat it as documentation, not code:

- Exclude the `wiki/` folder from your formatter (Prettier, Biome, etc.). Format conflicts and
  whitespace churn confuse the agent and bloat diffs.
- Exclude the `wiki/` folder from your type-checker / linter. If you use a per-module
  `tsconfig.json`, don't include `wiki/` in any of them.
- Don't apply your code-review automation (spell-check on identifiers, etc.) to wiki content.

The wiki is **not** a substitute for code comments where the WHY is non-obvious. If a single line
of code needs a comment, write the comment. The wiki is for context that spans files or sessions.
