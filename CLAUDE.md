# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A Claude Code **skill** repository holding one skill:

- `transcript-review/` — mine past Claude Code session transcripts for evidence of how a skill or workflow actually behaved in practice.

A skill is markdown, not executable code: YAML frontmatter declaring `name` and `description`, then the body as the instructions. The directory name matches the frontmatter `name`.

## No build, lint, or test tooling

There is no `package.json`, test runner, linter, or CI config. Do not look for npm scripts or a test command — none exist. Changes are validated by invoking the skill against a real transcript corpus.

## The code inside SKILL.md runs against transcripts, not this repo

The JavaScript in `transcript-review/SKILL.md` reads `~/.claude/projects/**/*.jsonl` — the user's own session history. It is the skill's payload, not this project's source, and there is nothing here for it to analyze. Evaluate it against the transcript format, not against this repository.

## The step order is load-bearing

The eight steps are sequenced so each one prevents a failure that would otherwise be silent — producing confident, wrong numbers rather than a visible error. Step 4 (isolating genuine human turns) is the one that decides whether any result means anything; step 3 (probing the schema first) exists because the fields step 4 relies on are undocumented internals. Do not reorder steps or drop step 3 to save effort.

## Verify the schema claims before trusting them

`userType`, `isMeta`, `isCompactSummary`, `toolUseResult`, and `isSidechain` are internal Claude Code transcript fields with no stability guarantee. The skill presents its filter as a hypothesis to re-verify for exactly this reason. If a change touches that filter, confirm the field behavior against current transcripts rather than reasoning from the documented example.

The concrete figures in the file (entry counts, inflation ratios) come from one 2026-08 corpus and are there to convey magnitude. Do not silently update them to match a new corpus without saying which corpus they describe.
