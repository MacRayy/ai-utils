# CLAUDE.md Seed

A single-file seed that bootstraps a project's main agent-instructions file (`CLAUDE.md`) by
discovery — detecting the stack (.NET, Node/TypeScript, Python, React, React Native, or a polyglot
monorepo mixing several) and filling an opinionated section skeleton from the repo's own manifests.

## What this is

`CLAUDE.md` is the file a coding agent reads first on every session in a project. A good one tells
the agent the stack, the layout, the exact commands to run before committing, the conventions that
lint can't enforce, and the traps the team has already been bitten by. A bad one is a generic
"be helpful and write clean code" that the agent ignores.

This seed doesn't ship a fixed `CLAUDE.md` — a useful one is too project-specific for that. Instead
it ships three things for the agent that reads it:

1. **A skeleton** (§A) — the section structure of a strong `CLAUDE.md`, with placeholders and
   inline guidance on what belongs in each section.
2. **Stack profiles** (§B) — per-stack cheat sheets (detection signals, default commands, layout
   norms, conventions, common traps) the agent uses to fill the skeleton.
3. **A discovery procedure** (§C) — how the agent inspects the repo and interviews the user to
   fill every placeholder with real, verified values.

The output is one file — `CLAUDE.md` at the repo root — tailored to the actual project.

## When to reach for this (and when `/init` is enough)

Claude Code's built-in `/init` already scans a repo and writes a `CLAUDE.md`. Be honest about where
this seed beats it, so you use it where it pays off:

- **Best fit — existing repos, especially polyglot / monorepo ones.** The value here is
  archaeology: pulling the *real* test/lint/build commands out of dozens of scripts, mapping module
  boundaries, spotting generated files. The per-stack profiles (§B) and monorepo grouping rules are
  exactly what `/init` is vaguest on. This is the case where the seed clearly wins.
- **Best fit — repos that also run `wiki-seed.md` and/or `agents-seed.md`.** This seed knows how to
  slot their sections in and keep `CLAUDE.md` short by pointing at the wiki (§D). `/init` knows
  nothing about those seeds.
- **Weaker fit — a fresh, single-stack project.** There's little to discover yet; you mostly get
  the section skeleton and command defaults. `/init` handles that about as well. The seed's real
  payoff on greenfield arrives later, once the repo has a layout and conventions worth capturing —
  at which point it's the "existing repo" case again.

**What it does NOT do:** invent the hard-won conventions only the team knows ("this helper is
obsolete", "always check `useX` before writing a formatter"). It produces a *correct-but-thin*
first draft — accurate on stack, commands, and layout — and relies on the §C interview plus normal
accretion (and the wiki) to fill the war stories over time. If you expected a file as lived-in as a
mature hand-written `CLAUDE.md` on the first pass, that's not what any generator can give you.

## How to use this file (for humans)

1. Copy `claude-md-seed.md` into the root of the target repo.
2. In a fresh agent session, paste:

   > Read `claude-md-seed.md` and follow its bootstrap instructions. Detect the stack, show me the
   > filled-in section outline and the commands you found before writing `CLAUDE.md`.

3. The agent inspects the repo, matches it to one or more stack profiles, and shows you a draft
   outline plus the commands it detected. Correct anything wrong — especially the **commands**
   (format / lint / typecheck / test / run) and the **traps**, which are the highest-value parts.
4. Approve, let it write `CLAUDE.md`.
5. Skim the result. The stack facts should be accurate on day one; the conventions and traps
   accrete over time as the team learns them.

If a `CLAUDE.md` already exists, the agent will read it, keep what's still true, and fold this
structure around it rather than clobbering it.

## How to use this file (for the LLM agent reading it)

When the user asks you to bootstrap from `claude-md-seed.md`:

1. **Run the discovery procedure in §C first.** Detect the stack(s), read the manifests, find the
   real commands, map the layout. Do NOT write anything yet.

2. **Match the repo to one or more stack profiles in §B.** A repo can hit several (a monorepo with
   a React front-end and a .NET back-end hits three). Pull the relevant rows from each.

3. **If a `CLAUDE.md` already exists**, read it in full. Preserve any project-specific truth it
   holds (conventions, gotchas, command names). Reshape it to the §A skeleton; don't discard
   content just because it isn't in the skeleton's order.

4. **Fill the §A skeleton** with verified values. Every command you list MUST come from a real
   script/target you found (package.json `scripts`, a `Makefile`, `*.csproj`/`Directory.Build.*`,
   `pyproject.toml`, `justfile`, etc.) — never invent `yarn typecheck` if no such script exists.
   Delete sections that don't apply to this project rather than leaving empty headers.

5. **Show the user the filled outline and the detected commands before writing the file.** The
   commands and the traps are where a wrong guess does the most damage — confirm them.

6. **Write `CLAUDE.md` at the repo root** (or the path the user names).

7. **Compose with the other seeds.** If `wiki-seed.md` and/or `agents-seed.md` were already run,
   their snippets are already in `CLAUDE.md` — keep them and slot them where the skeleton marks.
   If they haven't been run but the folders exist (`wiki/`, `.claude/agents/`), reference them. If
   neither, drop those two skeleton sections entirely and don't reference them. See §D.

8. **Do not delete `claude-md-seed.md`** unless the user asks. Leaving it documents the bootstrap.

---

## §A — The `CLAUDE.md` skeleton to create

This is the file to produce. It uses **four backticks** so the embedded three-backtick command
blocks render cleanly. `<PLACEHOLDER>` markers and `<!-- guidance -->` comments must all be
resolved or deleted before you write the file — no bracketed placeholders and no guidance comments
may remain in the output.

````markdown
# <PROJECT_NAME> — Claude Code instructions

<!-- One or two sentences: what this project is and who uses it. E.g.
"ActHub is a Next.js frontend for the Actuate IoT platform." Keep it factual, not marketing. -->

<!-- WIKI SLOT — if wiki-seed.md was run (a `wiki/` folder exists), keep its integration snippet
here. If not, delete this comment. See the seed's §D for the exact snippet. -->

<!-- PROTOCOLS SLOT — if agents-seed.md was run (`.claude/agents/` exists), keep its challenge
protocol sections here. If not, delete this comment. See the seed's §D. -->

## Project overview

<!-- The 30-second orientation. What the system does, what shape it is (single app? monorepo?
service + client?), and the one or two facts a new contributor most needs up front. -->

## Tech stack

<!-- Bullet list. Language + version, framework + version, package manager, test runner, and any
load-bearing library the agent must respect (state management, ORM, UI kit, API client). Pull
versions from the manifests, not from memory. For a polyglot repo, group by surface:
"Frontend:" / "Backend:" / "Mobile:". -->

- **Language:** <e.g. TypeScript 5.x / C# 12 (.NET 8) / Python 3.12>
- **Framework:** <e.g. Next.js 16 / ASP.NET Core / FastAPI / Expo>
- **Package manager:** <e.g. yarn 4 / npm / pnpm / NuGet / uv / poetry>
- **Testing:** <e.g. Vitest + Playwright / xUnit / pytest>

## Architecture

<!-- Keep this short if a wiki exists — point at `wiki/architecture.md` for the deep version and
only summarize here. Without a wiki, this is where the layout and layering rules live. -->

### Layout

<!-- A tree or flat list of the top-level modules/projects and one line on what each holds. For a
monorepo, list every workspace/project. For a single app, list the top-level source folders. -->

```
<repo-root>/
  <dir-or-project>/   <what lives here>
  <dir-or-project>/   <what lives here>
```

### Boundaries & dependency rules

<!-- The import/reference rules that, if violated, cause real harm: which layer may depend on
which, what must never be imported, where business logic belongs vs. where it must not. List
3–6 invariants, not aspirations. Delete this subsection if the project is small enough not to
have meaningful boundaries. -->

- <e.g. "UI components must not call the API directly — go through the api-client module.">
- <e.g. "Domain layer must not reference the web/controller layer.">

## Common commands

<!-- THE most load-bearing section. Every command here must be one you verified exists. Lead with
the pre-commit gate, because that's what the agent needs on every task. -->

**Always run before committing:**

```bash
<FORMAT_COMMAND>      # format
<LINT_COMMAND>        # lint
<TYPECHECK_COMMAND>   # type-check   (omit if the stack has no separate type-check step)
<TEST_COMMAND>        # unit tests
```

**Day-to-day:**

```bash
<RUN_COMMAND>         # start the app / dev server
<BUILD_COMMAND>       # production build
<TEST_E2E_COMMAND>    # end-to-end tests   (omit if none)
<CODEGEN_COMMAND>     # regenerate types/clients/migrations   (omit if none)
```

<!-- Add any other commands the agent will reach for often (storybook, db migrate, seed data,
open API docs). Every entry maps to a real script/target. -->

## Code standards

### Formatting & linting

<!-- Which tool owns formatting (Prettier / dotnet format / black+ruff / …) and where its config
lives. Name the settings a reader can't infer from the config file if they matter (quote style,
line width, import ordering). Note that cosmetics are the formatter's job — the agent shouldn't
hand-format. -->

### Types

<!-- Type discipline for this stack: "no `any` without justification", nullable-reference-types
on, `strict` mode, generated types that must not be hand-edited, etc. Delete if untyped. -->

### <Language / framework> conventions

<!-- The conventions lint doesn't catch but the team enforces. Pull the relevant ones from the
seed's §B stack profile and keep only the ones this project actually follows. Examples:
- naming (boolean prefixes, file naming, interface prefixes)
- component/module structure (colocate tests, one export per file)
- error handling and logging patterns
- state/mutation rules (functional style, immutability)
- styling approach (CSS modules, utility classes) -->

## Where things live

<!-- A quick "when working on X, look in Y" map. This saves the agent a grep on every task. -->

- **<New feature / route / endpoint>:** `<path>`
- **<Reusable components / shared code>:** `<path>`
- **<Utilities / helpers>:** `<path>`  ← check here before writing a new helper
- **<Tests>:** `<path or "colocated">`
- **<Config>:** `<paths to the real config files>`

## Development workflow

### Getting started

```bash
<INSTALL_COMMAND>     # install dependencies
<CODEGEN_COMMAND>     # generate types/clients   (if applicable)
<RUN_COMMAND>         # start developing
```

### Before making changes

1. **Read the existing code first** — match the style and structure of similar code.
2. **Reuse before you build** — grep the helper locations under "Where things live" before adding
   a new util / hook / component / service.
3. <Any project-specific pre-flight step, e.g. "regenerate types if the API schema changed".>

## Testing

<!-- Where each kind of test lives and how to run it. Unit vs. integration vs. e2e. What "done"
means for tests in this repo (behavior-first, no snapshot churn, contract tests for the API,
etc.). Delete kinds that don't exist. -->

## <Integration / API contract>

<!-- Only if the project has a non-trivial external contract: an OpenAPI-generated client, a
GraphQL schema, a message queue, a third-party API. Describe the flow (schema → codegen → typed
client → usage) and what must NOT be hand-edited. Delete this whole section if not applicable. -->

## Git workflow

- **Default branch:** `<DEFAULT_BRANCH>`
- **Branch naming:** `<e.g. feature/descriptive-name>`
- **Commit style:** <state the concrete rule. If the repo has a consistent existing convention
  (from `git log` or a `CONTRIBUTING.md` / commit-style doc), name it and say "follow the existing
  convention". If the history is inconsistent / ad-hoc, state: "history is inconsistent — use
  Conventional Commits (`type(scope): subject`) going forward". Point at the source of truth if
  one exists.>
- **No AI attribution in commits** — do not add a `Co-Authored-By: Claude` (or any AI) trailer,
  and don't mention Claude/the agent in the commit message. Commits are authored under the human's
  git identity only.
- Run the pre-commit commands (above) before every commit; keep commits atomic.

## Environment setup

<!-- Prerequisites (runtime versions, Docker, SDKs) and how env/secrets are configured
(`.env.local` from an example file, user-secrets, appsettings, etc.). Where the example lives.
Delete if there's nothing beyond "install and run". -->

## Best practices

**Do:**

- <3–6 stack-specific do's the team cares about.>

**Don't:**

- <3–6 don'ts — especially "don't edit generated file X", "don't add deps without asking",
  "don't skip type-check".>

## Notes for Claude

<!-- The short, direct reminders that set the tone for this specific codebase. E.g.
"This is a production codebase — prioritize maintainability. Check existing patterns before
implementing. Respect module boundaries. Don't use `any`. Code-quality tools are strict for a
reason — follow them." Keep it to a handful of lines. -->
````

---

## §B — Stack profiles

Reference cheat sheets. Match the target repo to one or more, then pull the relevant rows into §A.
These are NOT written into `CLAUDE.md` verbatim — they're the raw material you filter down to what
the project actually does. When a repo hits several profiles (polyglot monorepo), merge them and
group §A's stack/commands by surface.

Always prefer a command you find in the repo's own scripts over the defaults below. The defaults
are what to expect, not what to assume.

### Node / TypeScript (back-end or shared)

- **Detect:** `package.json`; `tsconfig.json`; lockfile picks the manager (`yarn.lock` → yarn,
  `pnpm-lock.yaml` → pnpm, `package-lock.json` → npm, `bun.lockb` → bun).
- **Commands:** read `package.json` `scripts`. Common: `<pm> run format` / `lint` / `typecheck` /
  `test` / `build` / `dev`. `<pm>` is `yarn` / `npm run` / `pnpm` / `bun`.
- **Layout norms:** `src/`, entry in `src/index.ts` or `src/main.ts`; tests colocated
  (`*.test.ts`) or under `test/`/`__tests__/`.
- **Conventions to consider:** ESM vs CJS; `strict` in tsconfig; no `any`; functional/immutable
  style; import ordering if a sort plugin is configured.
- **Traps:** `new Date('YYYY-MM-DD')` parses as UTC; `as Type` assertions hiding unsafe casts;
  editing generated files (`*.gen.ts`, OpenAPI output); floating promises; `process.env` reads
  without validation.

### React (front-end, web)

- **Detect:** `react` + `react-dom` in `package.json`; often a meta-framework (`next`, `vite`,
  `remix`, `react-router`, `gatsby`) and a UI kit (Radix, MUI, Chakra, shadcn).
- **Commands:** `<pm> run dev` / `build` / `lint` / `typecheck` / `test`; e2e via `playwright` /
  `cypress`; component docs via `storybook` if present.
- **Layout norms:** components with colocated tests/stories/styles; routes under the framework's
  convention (`app/` or `pages/` for Next, `src/routes/` etc.).
- **Conventions to consider:** functional components + hooks; boolean props default `false` and
  use `is/has/should` prefixes; import individual React items (`import { useState }`) not default
  React; CSS Modules / utility classes over inline style; keep business logic out of presentational
  components.
- **Traps:** unstable `key`s across renders; object/array refs in effect dependency arrays;
  prop-rest leak when spreading onto DOM elements; `useEffect` for derived state; server/client
  component boundaries (Next App Router).

### React Native / Expo (mobile)

- **Detect:** `react-native` in `package.json`; `app.json`/`app.config.*` and `expo` for Expo;
  `metro.config.js`; native folders `ios/` + `android/`.
- **Commands:** `<pm> run start` (Metro); `<pm> run ios` / `android`; Expo: `npx expo start`,
  `eas build`; tests via jest/vitest.
- **Layout norms:** shared components; platform-specific files (`.ios.tsx` / `.android.tsx`);
  navigation config (React Navigation / Expo Router).
- **Conventions to consider:** no web-only DOM APIs; platform checks via `Platform.select`;
  styling via `StyleSheet` / a RN styling lib, not CSS; asset handling.
- **Traps:** importing Node/DOM-only libs; `console.log` left in release; forgetting to rebuild
  after native-dependency changes; list performance (`FlatList` keys, `getItemLayout`); env/secrets
  bundled into the app.

### .NET (C#, back-end)

- **Detect:** `*.sln`, `*.csproj`, `Directory.Build.props`, `global.json`; ASP.NET Core via
  `Microsoft.NET.Sdk.Web`; EF Core via `Microsoft.EntityFrameworkCore.*`.
- **Commands:** `dotnet restore` / `dotnet build` / `dotnet test` / `dotnet run`
  (`--project <path>`); `dotnet format` (or `dotnet format --verify-no-changes` as the "check");
  EF: `dotnet ef migrations add` / `database update`. No separate type-check step — the compiler
  is the type-check, so omit `<TYPECHECK_COMMAND>` and note that `dotnet build` covers it.
- **Layout norms:** project-per-concern (`*.Api`, `*.Domain`, `*.Infrastructure`, `*.Tests`);
  `Program.cs` entry; clean/onion layering common.
- **Conventions to consider:** nullable reference types on; async all the way (`Task`/`await`,
  `CancellationToken` on I/O); DI via the built-in container; analyzers/`.editorconfig` as the
  style source of truth; records for DTOs.
- **Traps:** blocking on async (`.Result`/`.Wait()` → deadlocks); missing `CancellationToken`;
  EF migrations not committed with the model change; secrets in `appsettings.json` instead of
  user-secrets/env; `DateTime.Now` vs `DateTime.UtcNow`.

### Python (back-end / tooling)

- **Detect:** `pyproject.toml` (preferred), `requirements.txt`, `setup.cfg`, `Pipfile`; framework
  via deps (`fastapi`, `django`, `flask`); manager via `uv.lock` / `poetry.lock` / `Pipfile.lock`.
- **Commands:** format `black .` or `ruff format`; lint `ruff check` / `flake8`; type-check `mypy`
  / `pyright` (only if configured); test `pytest`; run `uvicorn app:app --reload` /
  `python -m ...` / `manage.py runserver`. Prefix with the runner if used (`uv run …`,
  `poetry run …`).
- **Layout norms:** `src/<pkg>/` or a top-level package; tests under `tests/`; entry in
  `main.py`/`app.py`/`manage.py`.
- **Conventions to consider:** type hints on public functions; `ruff`/`black` own formatting;
  virtualenv/lockfile discipline; pydantic models at API boundaries.
- **Traps:** mutable default args (`def f(x=[])`); `dict.get(k) or default` masking falsy values;
  bare `except:`; import-time side effects; untyped `**kwargs` at boundaries; committing `.env` or
  a virtualenv.

### Monorepo / polyglot notes

- **Detect:** multiple manifests across folders; `pnpm-workspace.yaml` / yarn workspaces /
  `nx.json` / `turbo.json` / `lerna.json`; a `*.sln` plus a `package.json`; several `pyproject.toml`.
- **Do:** list every workspace/project in §A's Layout with one line each. Group commands and stack
  by surface (Frontend / Backend / Mobile). If the repo has a root-level task runner (`turbo`,
  `nx`, `make`, `just`), prefer its aggregate commands (`turbo run lint`) for the pre-commit gate,
  and note per-package commands where they differ.
- **Boundaries:** monorepos live or die by their dependency rules — capture them in §A's
  "Boundaries & dependency rules" even if you keep everything else short.

---

## §C — Discovery procedure

Run this before writing anything. The goal is a `CLAUDE.md` whose every factual claim you verified,
not guessed.

1. **Inventory manifests.** Glob for `package.json`, `*.sln`, `*.csproj`, `pyproject.toml`,
   `requirements.txt`, `go.mod`, `Cargo.toml`, `app.json`, `Makefile`, `justfile`, `turbo.json`,
   `nx.json`. Their presence and location tells you the stack(s) and whether it's a monorepo.

2. **Match stack profile(s) in §B.** Usually one; a polyglot repo hits several.

3. **Extract the real commands.** This is the highest-value step. Read `package.json` `scripts`,
   `Makefile`/`justfile` targets, `pyproject.toml` `[tool.*]` and script entries, `*.csproj`
   targets. Map each to a skeleton placeholder (`<FORMAT_COMMAND>` …). If a step genuinely doesn't
   exist (e.g. no separate type-check in .NET/Python), delete that line — don't fabricate one.

4. **Determine the package manager** from the lockfile, not preference.

5. **Map the layout.** List top-level modules/projects and one line each on what they hold.
   Identify the dependency/boundary rules if the project is large enough to have them.

6. **Detect config sources of truth.** Where do formatting/lint/type rules live
   (`.editorconfig`, `eslint.config.*`, `.prettierrc*`, `pyproject.toml`, `Directory.Build.props`,
   `tsconfig.json`)? Point at them; don't restate every rule.

7. **Find the git conventions.** `git log --oneline -20` for the commit-message style; the default
   branch (`git symbolic-ref refs/remotes/origin/HEAD` or `git branch`). Determine the commit-style
   rule with this precedence:
   1. A `CONTRIBUTING.md` or dedicated commit-style doc — defer to it verbatim.
   2. Otherwise, a **consistent** convention visible in the history (Conventional Commits, gitmoji,
      a `[type]:` prefix, etc.) — record "follow the existing convention" and name it.
   3. Otherwise (history is ad-hoc / inconsistent / random subjects) — default to **Conventional
      Commits** (`type(scope): subject`) and say the file is *establishing* the convention going
      forward, not describing existing history.
   Note the existing conventions elsewhere (naming, structure, config-driven rules) follow the same
   spirit: match what the repo already does; only impose a default where the repo has none.

8. **Interview the user for what the repo can't tell you** — keep it short:
   - Conventions lint doesn't enforce (naming, structure, patterns the team cares about).
   - Known traps / footguns (seed §B has stack-typical ones as starting candidates).
   - Which files must never be hand-edited (generated clients, migrations, schema dumps).
   - Deps policy, branch policy, anything with a "we've been bitten by this" story.

9. **Fill §A, delete every unused placeholder and guidance comment, show the user the outline +
   commands, then write the file.**

---

## §D — Composing with the wiki and agents seeds

This seed is designed to sit alongside `wiki-seed.md` and `agents-seed.md`. All three patch or
create the same `CLAUDE.md`, so coordinate rather than overwrite.

- **Order — run this seed LAST.** If the user is running more than one, do `wiki-seed.md` and
  `agents-seed.md` first, then this one. This seed is the structural owner of `CLAUDE.md`: it reads
  the file the other two produced, reshapes everything into the full skeleton, and keeps their
  snippets in the right slots (per bootstrap step 7). It also wires cross-references that only work
  if the targets already exist — e.g. §A's Architecture section points at `wiki/architecture.md`
  only when a `wiki/` folder is present. Run it first and those links can't be made, and the other
  two seeds will insert their snippets "near the top" without knowing the skeleton's layout. Running
  it first still works, just less cleanly — it detects and preserves whatever the others add later.

- **Wiki slot.** If a `wiki/` folder exists, keep (or insert) the wiki integration snippet at the
  WIKI SLOT in §A. That snippet is owned by `wiki-seed.md` (its §C) — don't duplicate it here; if
  it's already present, leave it. If there's no wiki, delete the slot and trim §A's Architecture
  section to be self-contained instead of pointing at `wiki/architecture.md`.

- **Protocols slot.** If `.claude/agents/` exists, keep (or insert) the challenge-protocol sections
  at the PROTOCOLS SLOT. Those are owned by `agents-seed.md` (its §B). If there are no agents,
  delete the slot. When the agents are present they read their project-specific knobs from
  `.claude/checks.md` (also owned by `agents-seed.md`, its §C) — so `CLAUDE.md` should not duplicate
  helper locations / traps / forbidden-file lists inline; point at `.claude/checks.md` instead and
  keep a single source of truth.

- **No wiki, no agents.** The `CLAUDE.md` this seed produces still stands on its own — it just
  omits those two sections.

---

## Why a seeded `CLAUDE.md`

The difference between a `CLAUDE.md` that changes agent behavior and one that doesn't is
specificity. "Write clean, maintainable code" is noise — the agent already tries to. "Run
`dotnet format --verify-no-changes` and `dotnet test` before every commit; never call `.Result` on
a `Task`; DTOs are records in `*.Domain`" is signal — it encodes this project's actual gates and
traps.

The hard part is that the useful content is different for every repo, which is exactly why a fixed
template fails and a discovery-driven seed works. The skeleton fixes the *shape* (so nothing
important is forgotten); the stack profiles supply the *candidates* (so the agent knows what to
look for in a .NET vs. a React Native repo); the discovery procedure grounds every claim in the
*actual repo* (so the commands are real and the layout is true). What's left for the human is the
handful of things no repo scan reveals: the conventions and the war stories.

## Pairs well with

- **`wiki-seed.md`** — the deep, living knowledge base. `CLAUDE.md` stays short and points at
  `wiki/` for architecture, decisions, and postmortems.
- **`agents-seed.md`** — the critic/gate sub-agents and the challenge protocols that this
  `CLAUDE.md` hosts. Their project-specific knobs live in `.claude/checks.md`, not inline here.
- **`code-quality-tools/`** — the lint/format/type configs whose commands this `CLAUDE.md`'s
  "Common commands" section points the agent at.

## License

This seed is free to use, copy, and modify in any project. No attribution required.
