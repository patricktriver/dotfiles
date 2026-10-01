---
name: council
description: "Runs a five-perspective deliberation (Contrarian, First-Principles Thinker, Expansionist, Outsider, Executor) with anonymous peer review and a Chairman synthesis to produce a decisive recommendation on hard decisions. Triggers on phrases like 'ask the council', 'run this by the council', 'what does the council think', '/council', or requests for a structured second opinion on an architecture, strategic, business, financial, career, or ambiguously-framed decision. Opens with a complexity gate that skips or confirms the full process for simple/low-stakes questions, since a full run costs ~11 agent calls."
compatibility: Requires an agent tool capable of spawning parallel subagents (e.g. Claude Code's Agent/Task tool); no external services required
---

# Council

A multi-agent deliberation process for decisions that are genuinely hard to reason about
alone: real tradeoffs, easy to anchor on the first plausible answer, or a framing that
might itself be wrong. Five independent perspectives analyze the problem in isolation,
five independent reviewers anonymously grade all five analyses, and a Chairman synthesizes
the strongest conclusions into one decisive recommendation.

This is expensive (5 + 5 agent calls, each with real context) — **Step 0 always runs
first** and most low-stakes invocations should never reach Stage 1.

The problem statement is whatever follows the trigger phrase (e.g. everything after
`/council`, or the substance of "ask the council: ..."). If it's missing or too vague to
analyze, ask the user to state it in one or two sentences before doing anything else.

## Step 0 — Complexity gate (always run this first, never skip)

Classify the problem before spawning anything:

- **Trivial / low-stakes** (single defensible answer, low cost of being wrong, not a real
  tradeoff — e.g. "should this be snake_case or camelCase"): don't run the council even if
  the user explicitly asked for it. Say briefly why it doesn't need deliberation, give a
  direct answer, and offer to run the full council anyway if they still want it.
- **Clearly complex or high-stakes** (architecture decision with real tradeoffs, strategic
  / business / financial / career decision, or the user's framing of the problem itself
  looks questionable): proceed straight to Stage 1, no need to ask — the user invoking the
  council on a question like this is already sufficient intent.
- **Ambiguous** (not obviously either): ask via `AskUserQuestion` — offer "Run the full
  Council (5 perspectives + peer review + Chairman synthesis — ~11 agent calls, a few
  minutes)" vs. "Just give me a quick, direct answer instead." Respect whichever the user
  picks.

If code or a repository is involved and the answer depends on facts about it (current
architecture, existing patterns, constraints), gather that context yourself before Stage 1
— the five council members should reason from real facts, not guesses, and subagents won't
have your conversation's context.

## Step 1 — Five independent council members

Read `PROMPTS.md` in this skill's directory for the exact role prompts and required output
format — use them verbatim, substituting the problem statement (plus any repo context
gathered above) for `{PROBLEM}`.

Launch all five as `Agent` tool calls with `subagent_type: general-purpose`, **in a single
message** (five parallel tool_use blocks) so they run concurrently and none can see the
others' output. Each agent gets only its own role prompt — never mention the other four
roles or let one agent's prompt reference another's answer.

## Step 2 — Anonymize and shuffle

In your own context (no agent call): strip the role name from each of the five responses,
reorder them into a sequence that is not the original 1-2-3-4-5 role order and varies
between runs, and label them Response A through Response E. See `PROMPTS.md` for the exact
procedure. Keep a private mapping of label → role for your own use in Step 4 — the
reviewers must never see it.

## Step 3 — Five independent peer reviewers

Read the Stage 3 reviewer prompt template in `PROMPTS.md`. Launch five `Agent` tool calls
with `subagent_type: general-purpose`, **in a single message**, each given the identical
input: the original problem and all five anonymized responses (A–E). None of the five
reviewers should see any other reviewer's assessment.

## Step 4 — Chairman synthesis

No new agent call — do this synthesis yourself, in your own context, since you already
hold everything it needs: the original problem, all five Stage 1 analyses (role labels
restored now that independence no longer matters), and all five Stage 3 reviews.

Synthesize, don't vote:

- Note where multiple independent perspectives converged — that's your strongest signal.
- Note real disagreements and which side's reasoning is actually stronger, and why.
- Separate high-confidence conclusions from speculation.
- If a minority view (one perspective, or one reviewer) exposes a serious risk or
  opportunity the majority missed, preserve it explicitly rather than letting it get
  averaged away.
- Don't blend incompatible recommendations into a mushy compromise — pick a side when the
  evidence supports it.
- Where the user's original framing or assumptions look questionable (the First-Principles
  Thinker or the Outsider flagged this), say so plainly.

## Final output

Don't dump the raw agent transcripts. Return exactly this structure:

```
## Council Report

**Verdict**
<one direct sentence: what the council recommends>

**Why**
<3-5 bullets, the strongest supporting reasons>

**Where the Council Disagreed**
<the most important unresolved disagreement or competing interpretation — omit this
section entirely if the council genuinely converged; don't manufacture disagreement>

**Key Risk**
<the most important failure mode identified>

**Missed Opportunity**
<the most important upside or non-obvious angle identified>

**Next Step**
<one concrete, immediately actionable step>

**Confidence:** Low | Medium | High
```

The Next Step must be specific enough to act on immediately — not "investigate further."

If the user asks to see the underlying detail (individual perspectives, rankings, reviewer
notes), you have it all in context already — share it directly rather than re-running
anything.

## Behavior rules

- Optimize for genuinely different reasoning across the five perspectives, not five
  stylistic rewrites of the same answer — if two Stage 1 responses converge, that's a real
  signal, not a sign something went wrong.
- Never let a later agent see or rewrite an earlier agent's output within Stage 1 or Stage 3.
- Never reveal role identity to the Stage 3 reviewers.
- Don't let majority agreement override a well-argued minority point without engaging its
  reasoning in Step 4.
- The council advises. Don't modify code, files, or take any other consequential action
  based on its recommendation unless the user explicitly asks you to, separately.

## Test prompts

Use these to sanity-check the skill after any changes to it:

1. **Architecture decision** — "Ask the council: should we split our monolith data
   pipeline into microservices, or keep it as one deployable unit as we scale?"
   Expect the full process to run (clear architectural tradeoff).
2. **Strategic/business decision** — "Run this by the council: we're considering raising
   prices 20% next quarter to fund a new engineering hire — good idea?"
   Expect the full process to run, and expect real tension between the Expansionist/Executor
   and the Contrarian.
3. **Ambiguous framing** — "What does the council think about how to convince my manager
   to approve budget for a tool I want?" Expect the First-Principles Thinker and/or
   Outsider to challenge the framing itself (e.g. "is 'convince' the right goal, versus
   making the business case?") rather than every perspective accepting the premise.
4. **Gate check (should NOT trigger full council)** — "/council should I call this
   variable `is_active` or `active_flag`?" Expect Step 0 to classify this as trivial, skip
   Stage 1-4 entirely, and just answer directly with an offer to run the full council if
   still wanted.
