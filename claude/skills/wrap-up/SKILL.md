---
name: wrap-up
description: "End-of-session reflection. Use when Patrick runs /wrap-up or asks to wrap up, review, or reflect on the current Claude Code session. Reviews the conversation, git changes, and any referenced Linear issues to give terse feedback on Claude Code usage and developer takeaways, then routes durable learnings to memory, CLAUDE.md, or Notion after Patrick confirms."
---

# Wrap-Up

Run at the end of a session when Patrick invokes `/wrap-up` or asks to wrap up / reflect.

## What to analyze

1. The conversation itself — tool-use patterns, friction, wasted turns, approaches that existed but weren't used.
2. `git status` / `git log` / `git diff` for the current repo, if inside one — what actually got built or changed.
3. Any Linear issues explicitly referenced or touched in the conversation (Linear MCP) — to tie a learning back to a ticket. Don't do a general Linear sync here; standing Team Green awareness already lives outside this skill (see `reference_linear_green_team.md` memory).
4. If the session's work was in `~/data-core/`, check whether anything learned exposes a gap or drift in the semantic layer (`snowflake/schemachange/views/semantic/R__6*_sv_*.sql`) — a business rule the session uncovered that isn't reflected in a view, a definition that turned out stale, a join or filter that should live there instead of being reimplemented ad hoc. Only surface this if the session's actual work touched data these views cover; don't go hunting for unrelated drift.
5. Check whether anything learned this session is relevant to the `~/triver-data-architecture` knowledge base — a platform-wide architectural convention, a decision and its rationale, a domain-model detail, or a gap between what's documented and what's actually true. Read `PLAN.md` there for current structure/priorities before proposing where it fits. Only surface this if the session surfaced something genuinely KB-worthy (the *why*, not routine implementation or debugging detail) — don't go hunting for unrelated gaps.

## Step 1 — Reflect (terse, on-screen, no confirmation needed)

Two sections max, 3-5 bullets each, no preamble, no restating the session, no motivational padding:

**Claude Code usage** — concrete things that would have saved time or tokens this session: a skill or memory that existed but wasn't used, Bash used where a dedicated tool fit better, a risky action approved without enough context, work that could've been forked/parallelized. Omit this section entirely if nothing stands out rather than manufacturing filler.

**Developer takeaways** — non-obvious things learned about the codebase, system, or domain this session that are worth remembering.

Keep it scannable. This is feedback meant to be read in ten seconds, not a report.

## Step 2 — Propose saves, then confirm before writing anything

For each candidate learning, pick one destination and show Patrick a short preview list (item + destination) before writing:

- **Memory** (`~/.claude/projects/-Users-patrickbyrne/memory/`) — facts about Patrick, feedback on working style, project state, or references to external systems. Follow the existing type/format conventions in that directory (frontmatter, `**Why:**` / `**How to apply:**` structure, link related memories with `[[name]]`).
- **CLAUDE.md** — standing rules Claude should always follow, not tied to a specific project, date, or piece of work.
- **Notion** — narrative, team-shareable learnings. Append a dated `## YYYY-MM-DD` section to the "Claude Code Wrap-Up Log" page in the Data Repository database (look up the URL via reference memory rather than hardcoding it — memory can go stale).
- **Semantic layer** (`~/data-core/snowflake/schemachange/views/semantic/`) — when the learning is a business-logic gap or drift in an `SV_*` view itself, not documentation about it. Propose the specific view file and the change (new case, corrected join/filter, updated comment), but don't edit it as part of wrap-up — that's real pipeline code and belongs in its own reviewed change, not a reflection step.
- **triver-data-architecture KB** (`~/triver-data-architecture/`) — when the learning documents a platform-wide architectural convention, a decision and its rationale, or a domain-model detail (architect-level *why*, per the KB's own framing — not an implementation log). Check `PLAN.md` for which doc file it belongs in (e.g. `domains/`, `quality/`, `decisions/`) and propose the specific file + edit. Unlike the semantic layer, this is documentation, not pipeline code — apply the edit directly once Patrick confirms it.

Don't write anything until Patrick confirms the preview. If he edits or drops an item, respect that.

## Durability rule for anything saved

Every saved item must read cleanly cold, months later, with zero session context:

- State the fact/mechanism, not the session narration ("X uses Y because Z", not "today we found out X uses Y").
- If something is genuinely time-bound (a sprint, a deadline, WIP state), say so explicitly and date it — don't imply permanence for something temporary.
- Prefer the underlying reason over the specific incident — the incident fades, the reason doesn't.
- Skip anything already derivable from code, git history, or existing CLAUDE.md content.
