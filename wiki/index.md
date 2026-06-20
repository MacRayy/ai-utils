# Wiki index

Map of content. Pages carry their own one-line summary at the top; this index quotes it.

## Start here

- [`architecture.md`](architecture.md) — layered model + worked end-to-end example.
- [`glossary.md`](glossary.md) — domain vocabulary with code references.
- [`CLAUDE.md`](CLAUDE.md) — schema: when to update, how to write pages.
- [`_log.md`](_log.md) — chronological changelog of wiki updates.

<!-- As you add pages, group them under the headers below. Each entry should be:
     - [`path/to/page.md`](path/to/page.md) — **<title>** — <one-line summary copied from the page>
     Delete a section header if it's empty. Add new headers as new categories emerge. -->

## Apps

<!-- One page per runtime surface (web app, API service, CLI, mobile, etc.). -->

<!-- No runtime apps — this repo is content (seeds + configs), not a runtime. -->

## Features

<!-- User-visible capabilities, top-down: route → component → API. -->

<!-- No user-facing features. The repo's "capabilities" are the seeds themselves, documented in architecture.md. -->

## Decisions

- [`decisions/2026-06-20-config-baseline-scope.md`](decisions/2026-06-20-config-baseline-scope.md) — **Code-quality configs target generic TS + React; Next.js stripped** — Records why Next.js, yarn-berry, and obsolete rules were removed from the imported configs.

## Bugs

<!-- Postmortems for non-obvious bugs. Date-prefixed: YYYY-MM-DD-slug.md. -->

## Runbooks

<!-- Recurring operational pain points with known recovery paths. -->

## Tech debt

<!-- Accepted longer-lived debt with current-cost notes. -->

## Hacks

<!-- Workarounds with concrete "remove when" conditions. -->

## Integrations

<!-- External systems and the contracts you rely on. -->

## Workflows

- [`workflows/commit-style.md`](workflows/commit-style.md) — **Commit message style** — Gitmoji prefix + Angular-style `[type]: subject`; no Claude co-author trailer.
