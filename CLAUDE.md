# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A Claude Code **skill** repository holding one skill:

- `skills/transcript-review/` — mine past Claude Code session transcripts for evidence of how a skill or workflow actually behaved in practice.

A skill is markdown, not executable code: YAML frontmatter declaring `name` and `description`, then the body as the instructions. The directory name matches the frontmatter `name`.

## It is also a plugin and its own marketplace

The repo root doubles as the plugin, and `.claude-plugin/marketplace.json` lists it with `"source": "./"`. That self-reference is deliberate and verified — `claude plugin validate` resolves it — so do not "fix" it into a `plugins/<name>/` subdirectory without re-validating.

Three files now restate the same one-sentence description: the skill's frontmatter, `plugin.json`, and the marketplace entry. Change one, change all three.

Note that `repository` in `plugin.json` must be a **string**. The published example showing an object (`{type, url}`) fails validation; the validator is the authority here, not the docs.

## No build, lint, or test tooling

There is no `package.json`, test runner, linter, or CI config. Do not look for npm scripts or a test command — none exist. Changes are validated by invoking the skill against a real transcript corpus, plus `claude plugin validate .claude-plugin/marketplace.json --strict` for the manifests.

## The code inside SKILL.md runs against transcripts, not this repo

The JavaScript in `skills/transcript-review/SKILL.md` reads `~/.claude/projects/**/*.jsonl` — the user's own session history. It is the skill's payload, not this project's source, and there is nothing here for it to analyze. Evaluate it against the transcript format, not against this repository.

## The social card is generated, not hand-drawn

`assets/social-card.png` is a 1280×640 screenshot of `assets/social-card.html`, taken at that exact viewport. Edit the HTML and re-screenshot; do not retouch the PNG. The `file://` protocol is blocked in the browser tooling, so serve the directory over localhost first.

The card quotes the 3,384 → 1,335 figure. It carries "one 2026-08 corpus" in the caption for the same reason the skill does — if the numbers change, that label changes with them.

## The step order is load-bearing

The eight steps are sequenced so each one prevents a failure that would otherwise be silent — producing confident, wrong numbers rather than a visible error. Step 4 (isolating genuine human turns) is the one that decides whether any result means anything; step 3 (probing the schema first) exists because the fields step 4 relies on are undocumented internals. Do not reorder steps or drop step 3 to save effort.

## Verify the schema claims before trusting them

`userType`, `isMeta`, `isCompactSummary`, `toolUseResult`, and `isSidechain` are internal Claude Code transcript fields with no stability guarantee. The skill presents its filter as a hypothesis to re-verify for exactly this reason. If a change touches that filter, confirm the field behavior against current transcripts rather than reasoning from the documented example.

The concrete figures in the file (entry counts, inflation ratios) come from one 2026-08 corpus and are there to convey magnitude. Do not silently update them to match a new corpus without saying which corpus they describe.
