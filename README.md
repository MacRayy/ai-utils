# ai-utils

Reusable artifacts for bootstrapping new projects.

## Contents

### `ai-seeds/` — LLM bootstrap seeds

Single-file markdown seeds. Drop one into a target repo, point an LLM agent at it, and the agent
expands it into a working setup (subagent files, wiki scaffolding, etc.).

- **`wiki-seed.md`** — bootstraps a `wiki/` knowledge base: schema (`wiki/CLAUDE.md`), page
  templates for decisions / bugs / hacks / tech-debt / runbooks / features / integrations /
  workflows, a `_log.md` changelog, and an integration snippet for the host project's main
  agent-instructions file.
- **`agents-seed.md`** — bootstraps six Claude Code subagents (`architect`, `plan-critic`,
  `solution-critic`, `reviewer`, `git-precommit`, `git-organizer`) plus three protocol sections
  for the host project's `CLAUDE.md`: plan-mode challenge, implementation challenge, optional
  pre-PR architectural self-check. Pairs with `wiki-seed.md`; works standalone too.

To use a seed in a fresh agent session:

> Read `<name>-seed.md` and follow its bootstrap instructions. Show me what you're about to
> create before writing any files.

For `agents-seed.md`, the agent will walk a small set of customization seams (default branch,
format/lint/typecheck/test commands, helper locations, forbidden files, project traps) with you
before writing.

### `code-quality-tools/` — copy-in baseline configs

Plain config files for TypeScript + React projects. Copy to a new repo's root and adapt for the
project's layout.

- **`eslint.config.mjs`** — flat-config ESLint with `typescript-eslint` strict + stylistic
  type-checked, React + react-hooks + jsx-a11y, import ordering, Prettier integration, and
  relaxed rules under `__tests__/` and `*.test.*`.
- **`prettier.config.js`** — 2-space indent, single quotes, no semicolons, 100-char line,
  trailing commas, `lf` line endings.
- **`tsconfig.json`** — strict-mode TS baseline (`ES2024`, `bundler` resolution, `react-jsx`,
  `noEmit`). Add your own `include` and `paths` for your source layout.

## License

MIT — see `LICENSE`.
