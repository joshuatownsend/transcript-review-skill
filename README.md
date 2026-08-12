# transcript-review

A [Claude Code](https://claude.com/claude-code) skill for mining your own past session
transcripts for evidence of how a skill or workflow *actually* behaved — as opposed to how
you remember it behaving, or how its instructions say it should.

Point it at a skill you maintain and it will find the sessions where that skill really ran,
pull the verbatim exchanges where it fell short, and separate what the evidence supports
from what it doesn't.

## Why it exists

Claude Code writes every session to `~/.claude/projects/**/*.jsonl`, so the raw material for
"did this skill work?" is already sitting on disk. The trouble is that the obvious way to
analyze it is wrong in ways that don't announce themselves.

Filtering on `type === 'user'` looks like it gives you the human's turns. It doesn't — in one
session from the reference corpus, 2,704 of 2,866 user-role entries were tool results. Adding
a text heuristic on top still admits task notifications, compaction summaries, and messages
from other agents, all of which read like prose. Across that corpus the naive count showed
3,384 human turns where 1,335 were real: a 2.5x inflation that produces a clean-looking report
built on nothing.

Grepping for a skill's name has the same character of failure. The list of installed skills is
injected into every session's context, so the name matches sessions that merely *had* it
available, not sessions that used it.

Neither mistake throws an error. Both yield confident, specific, wrong numbers. The skill is
structured around catching them.

## What it does

`transcript-review/SKILL.md` is the whole thing — eight steps covering where transcripts live,
how to process them without burning context, how to verify the schema before trusting a filter,
how to isolate genuine human turns, how to detect real invocations, which signals are worth
counting, and how to report findings honestly.

**The step order is load-bearing** — each step exists to prevent a failure the next one would
otherwise inherit silently — so this README deliberately doesn't restate the steps as a
checklist. Read `transcript-review/SKILL.md` for the actual instructions.

Two design choices worth knowing before you use it:

- **It counts what the user did, not what they said.** Complaints are rare and unreliable as a
  signal; people don't argue with an agent, they just work around it. The skill instead looks
  for turns where the human did the job the skill should have done — hand-timing an invocation,
  prompting a re-check, relaying a signal the agent should have read itself, contradicting a
  claim the agent made. Those are unambiguous.
- **Transcripts are evidence about the past.** They can validate that a problem was real; they
  can never validate that your fix worked, because the data predates the fix. The skill is
  required to disclose which version of a skill the data actually measured.

## Install

Skills are directories containing a `SKILL.md`. Copy or symlink `transcript-review/` into
either location:

```bash
# personal — available in every project
git clone https://github.com/joshuatownsend/transcript-review-skill.git
cp -r transcript-review-skill/transcript-review ~/.claude/skills/

# or per-project
cp -r transcript-review-skill/transcript-review .claude/skills/
```

Then invoke it as `/transcript-review`, or just ask for the work in your own words — the
frontmatter `description` lets Claude pick it up from phrasing like "review my session
transcripts" or "how has this skill actually been performing?"

## Usage

Typical asks:

- *"Review my transcripts for how the pr-resolve skill has been behaving."*
- *"Find the friction in how I've been doing releases — anything worth codifying into a skill?"*
- *"Where have I been correcting the agent about the same thing repeatedly?"*

Expect a report that leads with quoted exchanges (dated, with the project they came from) and
keeps aggregates as support, followed by proposed edits for you to accept or reject. The skill
is instructed to check its interpretation with you before building on it, and to disclose which
categories were mostly regex noise.

## Caveats

- **It reads undocumented internals.** `userType`, `isMeta`, `isCompactSummary`,
  `toolUseResult`, and `isSidechain` are Claude Code transcript fields with no stability
  guarantee, and they can drift between versions. The skill presents its filter as a hypothesis
  to re-verify, and step 3 exists specifically to make you check before trusting it. If it ever
  starts returning implausible numbers, that's the first place to look.
- **The figures are from one corpus.** The entry counts and inflation ratios quoted above and in
  `SKILL.md` come from a single machine's transcript corpus as of 2026-08. They're there to
  convey magnitude, not to be current truth about your data.
- **Nothing leaves your machine.** All analysis runs locally against `~/.claude/projects`. The
  skill also enforces that transcripts are processed *in code* with only derived summaries
  printed — partly to protect context, and partly because a routine machine holds several
  thousand transcript files and you do not want them in a conversation.

## Repository layout

```
transcript-review/
  SKILL.md      # the skill itself — frontmatter plus instructions
CLAUDE.md       # guidance for Claude when editing this repo
```

There is no build, test, or lint step; a skill is markdown. Changes are validated by running
the skill against a real transcript corpus.
