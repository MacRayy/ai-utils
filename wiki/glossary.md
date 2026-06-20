# Glossary

> Domain terms used across this repo and its seeds.

## Seed

A single self-contained markdown file (under `ai-seeds/`) that an LLM agent reads and expands into a working setup in a target repo. Each seed has three parts: §A files-to-create, §B folders-to-scaffold, §C integration snippet for the host project's agent instructions.
**Code:** `ai-seeds/wiki-seed.md`, `ai-seeds/agents-seed.md`.

## Host project

The downstream repo a seed is dropped into. The seed and its expanded files live in the host project; `ai-utils` is the source of seeds, not their consumer.

## Customization seams

The bracketed placeholders (`<DEFAULT_BRANCH>`, `<HELPER_LOCATIONS>`, `<PROJECT_TRAPS>`, etc.) in `agents-seed.md` that the host agent must fill in **with the user** before writing any files. The user walks the seams with the agent so the produced subagents are tuned to the host project's stack.
**Code:** `ai-seeds/agents-seed.md` §C.

## Critic (subagent)

A Claude Code subagent whose only job is to push back on work the main agent has produced. The four in `agents-seed.md` — `architect`, `plan-critic`, `solution-critic`, `reviewer` — each ask a different question and default to `"needs_revision"` so approval feels earned.
**Related:** [[Blindfolded architect]].

## Blindfolded architect

The `architect` subagent's deliberate restriction: it reads tests + wiki only, never source. The blindfold prevents it from anchoring on the existing implementation when scouting alternatives. The active ingredient — if it just echoes existing patterns it has failed.
**Code:** `ai-seeds/agents-seed.md` (the `architect` definition block).

## Wiki schema

The `wiki/CLAUDE.md` page that defines *when* to update the wiki and *how* to write each page type. The schema, not any individual page, is the source of truth for taxonomy. When content stops fitting an existing category, evolve the schema first, then add the page.
**Code:** `wiki/CLAUDE.md`.

## Field note

A longer-form write-up under `field-notes/` capturing the design rationale, tradeoffs, or retrospective for something built in this repo. Distinct from a `decisions/` page: field notes are narrative and write-once; decisions are structured and have a status (proposed/accepted/superseded).
**Code:** `field-notes/2026-06-18-architect-agent.md` is the first example.
