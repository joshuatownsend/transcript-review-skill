---
name: transcript-review
description: Mine Claude Code session transcripts for evidence of how a skill or workflow actually behaved — review session transcripts, analyze how a skill performed, find friction to codify into a new skill
---

Evidence for how a skill really behaves lives in past session transcripts. The
work is not hard, but three of its failure modes produce *confident, wrong
numbers* rather than obvious errors — so follow the order below. Each step
exists because skipping it silently corrupts the result.

## 1. Locate the transcripts

`~/.claude/projects/<slug>/<session-id>.jsonl`, one JSON object per line. The
slug is the project's working directory with separators replaced by `-`
(`C:\dev\myapp` → `C--dev-myapp`). **Match slugs case-insensitively** — the same
project can appear as both `C--dev-myapp` and `c--dev-myapp`.

Subagent transcripts sit at `<session-id>/subagents/agent-*.jsonl`. **Exclude
them when analyzing human behavior**: their user-role turns are the parent
agent's prompts, not a person's. Entries also carry `isSidechain` for this.

## 2. Never read transcripts into context

The corpus is far larger than it looks — a routine machine holds several
thousand files and single sessions run past 25,000 lines. Process them in code
and print only derived summaries. `Read` on a transcript, or any command that
dumps matches, spends context on raw bytes and buys nothing.

Use a sandboxed runner if one is available (`ctx_execute` with `language:
"javascript"`), otherwise `Bash` with `node -e` writing to the scratchpad. The
invariant is what matters, not the tool: **derive in code, print conclusions.**

On Windows, do not run recursive `du`, `find -exec`, or size sweeps over the
projects tree — they routinely exceed a two-minute timeout. Walk the tree inside
your script instead.

## 3. Probe the schema before trusting any filter

The fields below are undocumented internals and can drift between Claude Code
versions. Before relying on them, dump the entry shapes from one large session
and confirm they still separate cleanly:

```javascript
// group user entries by shape; eyeball which combination is human-typed
const k = `content=${ct} toolUseResult=${!!o.toolUseResult} ` +
          `userType=${o.userType} isMeta=${o.isMeta} isCompact=${o.isCompactSummary}`;
```

Treat step 4 as a starting hypothesis you re-verify, not a constant.

## 4. Isolate genuine human turns

This is the step that decides whether the analysis means anything. Filtering on
`type === 'user'` alone is badly wrong: in a representative session, 2,704 of
2,866 user-role entries were tool results. Text-prefix heuristics are also
insufficient — they let through task notifications, compaction summaries, and
teammate messages, all of which read like prose and inflate every count.

```javascript
function human(o) {
  if (o.type !== 'user' || o.userType !== 'external') return null;
  if (o.toolUseResult || o.isMeta || o.isCompactSummary) return null;
  let t = typeof o.message.content === 'string' ? o.message.content
        : (o.message.content || []).filter(c => c.type === 'text')
            .map(c => c.text || '').join('\n');
  t = t.replace(/<system-reminder>[\s\S]*?<\/system-reminder>/g, '').trim();
  if (!t || t.length < 4) return null;
  if (/^<|^PS [A-Z]:|^Another Claude session|^Caveat:|^\[Request interrupted/.test(t))
    return null;
  return t.replace(/\s+/g, ' ');
}
```

The prefix rejections each correspond to a real pollutant: `<task-notification>`,
`<bash-stdout>`, `<local-command-stdout>`, `<command-name>`, `<teammate-message>`
(another agent, not the user), pasted PowerShell sessions, and compaction
summaries beginning "This session is being continued from a previous
conversation." Applying this filter to one corpus reduced 3,384 apparent human
turns to 1,335 real ones — a 2.5x inflation that would have gone unnoticed.

## 5. Find invocations, not mentions

The list of available skills is injected into **every** session's context, so
grepping for a skill's name matches sessions that merely had it installed. Match
on actual use instead:

- an assistant `tool_use` where `name === 'Skill'` and `input.skill` matches
- a user turn containing `<command-name>/<skill-name></command-name>`

## 6. Choose a signal that means something

Counting complaints does not work — people rarely complain to an agent. The
productive question is **where did the user do the job the skill should have
done?** Those turns are unambiguous and easy to match:

- **hand-timing** the invocation — "run it once the bots review" means the skill
  does not know when to start
- **prompting a re-check** — "check again" means it stopped too early
- **relaying a signal** the agent should have read itself
- **contradicting a claim** — agent says clean, user says otherwise within a few
  turns; pair each claim with the next human turn to catch these

Pull verbatim quotes with dates and projects. A single quoted exchange where the
agent asserted something false and the user corrected it is worth more than any
aggregate.

## 7. Validate before concluding

Raw frequencies mislead. Before reporting a number, ask what else produces it:
invocation counts are not a failure rate, because one session legitimately runs
a skill against several targets. State the confound or lead with cleaner
evidence, and **check the interpretation with the user before building on it.**

Two disclosures belong in every report: which categories were mostly regex noise,
and what version of the skill the data actually measured. Transcripts are always
evidence about the *past* version — they validate problems, never fixes.

## 8. Report

Separate what the evidence supports from what it does not: gaps already
addressed, gaps still open, and known-weak categories. Lead with verbatim
exchanges; keep aggregates as support. Then propose specific edits and let the
user choose which to apply.
