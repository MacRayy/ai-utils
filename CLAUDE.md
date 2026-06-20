# ai-utils

A collection of reusable artifacts for bootstrapping new projects: single-file **seeds** that an
LLM agent expands into a working setup, and **code-quality-tool configs** (ESLint, Prettier,
TypeScript) meant to be copied into target repos as a baseline.

## What's here

### `ai-seeds/` — LLM bootstrap seeds

Self-contained markdown. A human drops the file into a target repo; an LLM agent reads it and
follows the embedded bootstrap instructions to generate the actual files.

- `ai-seeds/wiki-seed.md` — bootstraps a `wiki/` knowledge base (schema + page templates +
  empty subfolders + integration snippet for the host project's agent instructions).
- `ai-seeds/agents-seed.md` — bootstraps six Claude Code subagents (`architect`, `plan-critic`,
  `solution-critic`, `reviewer`, `git-precommit`, `git-organizer`) plus three protocol sections
  for the host project's `CLAUDE.md`. Pairs with the wiki seed; degrades gracefully without it.

### `code-quality-tools/` — copy-in lint / format / type-check baseline

Plain config files, not seeds. Drop into the root of a new TypeScript + React project and adapt
to its layout (paths, includes, package-manager-specific ignores).

- `code-quality-tools/eslint.config.mjs` — flat-config ESLint for TypeScript + React + Prettier:
  `typescript-eslint` strict + stylistic type-checked, React + react-hooks + jsx-a11y, import
  ordering, Prettier integration, relaxed rules under `__tests__/` and `*.test.*`.
- `code-quality-tools/prettier.config.js` — pure formatting preferences (2-space, single quotes,
  no semicolons, 100-char line, `lf` line endings, trailing commas).
- `code-quality-tools/tsconfig.json` — generic strict-mode TS baseline (`ES2024`, `bundler`
  resolution, `react-jsx`, `noEmit`). Consumers must add `include` and any `paths` aliases for
  their own source layout.

## How a seed is structured

Every seed follows the same three-part shape:

1. **Human-facing intro** — what the seed does, when to use it.
2. **LLM bootstrap instructions** — step-by-step for the agent reading the file.
3. **§A / §B / §C sections** — files to create, folders to scaffold, snippets to inject into the
   host project's main agent-instructions file. File contents are fenced with **four backticks**
   so embedded three-backtick examples render cleanly.

`agents-seed.md` adds a **§C — Customization seams** section with bracketed placeholders
(`<DEFAULT_BRANCH>`, `<HELPER_LOCATIONS>`, `<PROJECT_TRAPS>`, etc.) that the host agent fills in
with the user before writing files. Walk these with the user first; don't write files with
placeholders still in them.

## Working on seeds in this repo

- Seeds are **content**, not code. There is no build, no test runner, no lint. Edits are reviewed
  by reading the markdown.
- Preserve the four-backtick fencing inside seeds. Embedded examples need three-backtick fences;
  the outer file uses four.
- Keep the human-facing intro and the LLM bootstrap instructions in sync — if you change what
  files get created, update both.
- The two seeds are designed to **compose**: `agents-seed.md` references `wiki/` paths
  (`wiki/index.md`, `wiki/decisions/`, `wiki/hacks/`, `wiki/tech-debt/`, `wiki/_log.md`) that
  come from `wiki-seed.md`. When editing one, check whether the other still lines up.
- Seeds are meant to be dropped into **target repos**, not run from this one. Don't try to
  "execute" a seed against `ai-utils` itself.

## Adding a new seed

A new seed should:

1. Live as a single file under `ai-seeds/<name>-seed.md`.
2. Open with what it bootstraps and when to use it (human-facing).
3. Include an explicit "how to use this file (for the LLM agent reading it)" section with
   numbered bootstrap steps.
4. Use four-backtick fences for any block that contains a file's full contents.
5. Document customization seams in their own section if the seed has any.
6. End with a short "why this matters" or "pairs well with" pointer to related seeds.
