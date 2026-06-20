# Code-quality configs target generic TS + React; Next.js stripped

> The imported `code-quality-tools/` configs were trimmed to a framework-agnostic TS + React baseline. Next.js, yarn-berry, monorepo paths, and one obsolete rule were removed.

**Date:** 2026-06-20  **Status:** accepted

## Context

The initial drop of `code-quality-tools/eslint.config.mjs` and `tsconfig.json` came from a Next.js monorepo. Both contained project-specific bits:

- ESLint: `@next/eslint-plugin-next` + its `recommended` and `core-web-vitals` rule spreads, `.next/*` and `next-env.d.ts` in `ignores`, `.yarn/*` (yarn-berry zero-installs) in `ignores`.
- ESLint: `'@typescript-eslint/interface-name-prefix': 'off'` — a rule removed from `@typescript-eslint` upstream years ago; toggling it does nothing.
- tsconfig: `paths` aliasing `@/*`, `application/*`, `api-client`, `core`, `ui-library` to `../../../modules/...` — a specific monorepo layout that resolves to nothing in this repo.
- tsconfig: `include` array pointing at the same `../../../modules/...` paths.
- tsconfig: `**/.next/**` in `exclude`.

These configs are reusable artifacts dropped into new projects, not configs run against `ai-utils` itself. Single-framework / single-monorepo bits make them less reusable.

## Options considered

- **A. Keep everything.** Maximally featured out of the box. Cons: forces every consumer to either be on Next.js + the same monorepo layout or strip these bits themselves; `tsconfig.json` errors immediately because the included `../../../modules/...` paths don't resolve.
- **B. Strip everything framework-tied (React + Next.js).** Pure TS-only baseline. Cons: React is the dominant frontend framework; pruning it makes the config significantly less useful for the common case.
- **C. Strip Next.js and the monorepo-specific bits. Keep React, hooks, jsx-a11y, TypeScript strict.** A useful baseline for any TS + React project regardless of bundler.

## Decision

**C.** The configs target "generic TS + React with a bundler" — the common case for new web projects.

## Consequences

- Consumers on Next.js can re-add `@next/eslint-plugin-next` themselves; everything else still applies.
- Consumers without React would prune `eslint-plugin-react` + `react-hooks` + `jsx-a11y` — still a smaller diff than adding everything back.
- `tsconfig.json` now ships with no `include`/`paths`; consumers must add their own per project layout. The `compilerOptions` (strict mode, ES2024 lib, bundler resolution, react-jsx, noEmit) stand alone.
- The README and root `CLAUDE.md` describe the configs as "TS + React baseline" so the scope is explicit.

## References

- Commit `eeda7b8` (the scaffold commit that landed the trimmed configs).
- The pre-trim state lived only in the index briefly; the cleanup happened before the first real commit.
