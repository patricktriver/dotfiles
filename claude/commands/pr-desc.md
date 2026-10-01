---
description: Generate a terse PR title and description from the diff and update the PR on GitHub
allowed-tools: Bash(git diff *), Bash(git log *), Bash(gh pr view *), Bash(gh pr edit *), Bash(gh pr list *)
---

!`git log main..HEAD --oneline`
!`git diff main...HEAD --stat`
!`gh pr view --json number,title,body,url 2>/dev/null || echo "no PR for current branch"`

You are the author. Write as if you (the user) wrote it. Terse, direct, no fluff.

Title rules:
- One line, under ~70 chars, imperative mood ("Fix", "Add", "Remove" — not "Fixed"/"Adds").
- Describes the effect, not the mechanics. No ticket IDs unless the existing title already had one (preserve that prefix/convention).
- No trailing period. No emojis.
- If the current title already captures the change well, keep it and say so — don't churn a good title for the sake of changing it.

Format — exact headers, in this order:
## What

## Why

## Testing

Rules:
- ## What = 1–3 bullets, the shape of the change. Not a commit log.
- ## Why = 1–2 bullets, the reason. Business driver or bug. Not "to improve code quality". If the bug was caught by noticing a mismatch against another source of truth (a manual calc, a dashboard, another system), say that plainly — it's often the most important sentence in the PR.
- ## Testing = bullets of what was tested and how. Manual steps if relevant. Mark untested areas honestly.
- Stay broad and non-technical: describe behavior and reasoning in plain language, not implementation. Never name specific functions, methods, classes, variables, or internal system/table names — the reader can open the diff for that level of detail. Say "the wrong data source" or "an internal store," not the class or field that implements it. If a broader, less precise term conveys the same point, prefer it over the exact technical one.
- No emojis. No "🤖 Generated with Claude Code". No "Co-Authored-By". No AI attribution whatsoever.
- Human voice: contractions OK, sentence fragments OK. Avoid marketing adjectives ("robust", "comprehensive", "seamless").
- If the diff is trivial, a one-liner per section is correct — don't pad.
- No restating-the-diff filler. The reader can see the diff. Skip things like "renamed foo.py to bar.py", "added import for X", "bumped version from 1.2 to 1.3", "moved function from A to B". Only mention a rename/move/bump if the *reason* matters (e.g. breaking change, security patch). Describe intent and effect, not mechanics.

Flow:
1. Draft a first pass from the diff alone — both the title and the body.
2. Decide whether questions are warranted. For a trivial PR (typo, one-line fix, dependency bump, obvious refactor) skip questions entirely — the diff speaks for itself. Only ask when the diff leaves real gaps a reader would care about. Calibrate the number to the PR's weight: 0 for trivial, 1–2 for moderate, up to 4 for substantial changes. Never ask for the sake of asking.
3. If asking, show the draft first, then pick only the themes that are genuinely unclear from these:
   - **Context** — what triggered this work? (ticket, incident, conversation, upstream change)
   - **Things you considered** — alternatives weighed and rejected, why this approach won
   - **Trade-offs made** — what got sacrificed (perf, simplicity, coverage, scope) and why that was acceptable
   - **Things you did do** — non-obvious work that doesn't show clearly in the diff (manual data fixes, infra changes, coordination)
   - **Things you didn't do (but may do later)** — known follow-ups, deferred cleanup, scope cuts
   Use AskUserQuestion when the answer is likely a choice between known options; otherwise ask in plain text.
4. Fold any answers into the draft. Surface trade-offs and deferred work as bullets in ## Why or a short ## Notes section if they don't fit cleanly elsewhere — don't bloat ## What.
5. Show the (possibly revised) title and body and ask for confirmation or edits.
6. On confirmation, run gh pr edit --title "..." --body "$(cat <<'EOF' ... EOF)" to update both the title and description in one call.
7. If no PR exists for the current branch, say so and stop — do NOT create one unless asked.
