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
- **`agents-seed.md`** — bootstraps a set of Claude Code subagents (`architect`, `plan-critic`,
  `solution-critic`, `reviewer`, `git-precommit`, `git-organizer`, `trap-learn`, and an optional
  `sensitive-data-auditor`) plus protocol sections for the host project's `CLAUDE.md`: plan-mode
  challenge, implementation challenge (with a behavioral **verify** step and a **trap-learn**
  self-improvement step), an optional pre-PR architectural self-check, and an optional **enforcement
  hook** that blocks `ExitPlanMode` until `plan-critic` approves. All project-specific
  knobs live in one `.claude/checks.md` the agents read at runtime — so the agents stay generic
  (shippable as a team plugin) and the critic loop can teach itself via `trap-learn`. Pairs with
  `wiki-seed.md`; works standalone too.
- **`claude-md-seed.md`** — bootstraps the host project's main `CLAUDE.md`. Detects the stack
  (.NET, Node/TypeScript, Python, React, React Native, or a polyglot monorepo), pulls the real
  format/lint/typecheck/test/run commands out of the repo's own manifests, and fills a section
  skeleton — so the result is grounded in the actual project, not a generic template. Composes
  with the wiki and agents seeds; works standalone.
- **`merge-risk-seed.md`** — bootstraps a risk-based merge-review gate
  ([`attrecto/merge-risk`](https://github.com/attrecto/merge-risk)): generates a `.merge-risk.yml`
  tuned to the repo's real risk surface via a stack-aware risk survey (auth / token / secrets /
  migrations / money / crypto-certs / infra → `very-risky`; docs / i18n / tests / stories / assets →
  `safe`; everything else → `risky`), validates the scoring, and wires the GitHub Action or GitLab CI
  job — with the three-tier model (and why not four), the approval-count dial, and the traps that bite
  everyone first (GitLab status-name collision, masked-not-protected token, base-branch bootstrap,
  approval-doesn't-re-run). The deterministic companion to the `agents-seed.md` critic loop.

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

## Recommended companion tools

### [rtk](https://github.com/rtk-ai/rtk) — command-output compression for the agent loop

The agents from `agents-seed.md` are bash-heavy: `git-precommit`, `solution-critic`, and `reviewer`
all run `git diff`, lint, typecheck, and the test suite, and most of that output is noise the model
pays tokens to read. **rtk** sits on the bash tool call and filters, groups, dedupes, and truncates
that output before the agent sees it — typically **60–90% fewer tokens** on exactly the commands
this loop runs the most. Built-in tools (`Read`, `Glob`, …) are untouched.

It's the cheapest way to make "run several critics on every task" affordable. Install once,
globally — it's not a per-repo artifact:

```bash
brew install rtk          # macOS/Linux (or: winget install rtk-ai.rtk)
rtk init -g               # install the Claude Code hook (use --agent cursor / --gemini for others)
rtk gain                  # see how many tokens it's saving you
```

Single Rust binary, <10 ms overhead, no command/file content collected. Pairs naturally with the
critic loop: the gates get much cheaper without changing a line in any agent.

## License

MIT — see `LICENSE`.
