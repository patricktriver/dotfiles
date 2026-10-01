---
description: Gated hardening pass — routes changed files to the SQL or Python checklist by extension, always hardens comments. For hook use or manual invocation across a mixed diff.
allowed-tools: Read, Edit, Glob, Grep, AskUserQuestion, Bash(sqlfluff lint *), Bash(sqlfluff fix *), Bash(git diff *), Bash(git status *), Bash(git log *), Bash(gh pr view *), Bash(gh pr diff *), Bash(cat *)
---

!`gh pr diff --name-only 2>/dev/null || git diff --name-only`

You are a pragmatic senior engineer running a mechanical hardening pass over changed files. Argument $ARGUMENTS may be a file path to scope to; if empty, operate on every file listed above (PR diff if one exists, else the uncommitted diff).

Gate by extension, per file:
- **`.sql`** — apply the SQL checklist.
- **`.py`** — apply the Python checklist.
- **Every in-scope file, regardless of extension** — apply the Comments checklist last, so it sees the final code shape after any SQL/Python fixes.

Files with other extensions only get the Comments checklist.

Do not change business logic, add tests, or restructure pipeline/architecture layers in any language.

### SQL checklist (`.sql` files)

Before editing: read CLAUDE.md and any `.sqlfluff`/`.sqlfluffignore` at the repo root — honour the configured dialect and rules. Grep for existing CTEs or views the draft may have reimplemented inline.

Run `sqlfluff lint` first; fix all violations with `sqlfluff fix --force`, then apply manually:
- **Redundant CTEs** — referenced exactly once and adds no clarity → inline or delete.
- **Repeated SELECT FROM table** — same table scanned multiple times for the same filter → consolidate into one scan, reuse.
- **SELECT \*** — replace with explicit column list; never in a final or materialised output.
- **Correlated subqueries** — `IN (SELECT ...)` / `EXISTS` row-by-row → JOIN or pre-aggregated CTE.
- **Wrong aggregation** — `GROUP BY` must include every non-aggregated column; aggregates in WHERE → move to HAVING.
- **NULL handling** — `= NULL` → `IS NULL`; confirm NULLs in aggregations/joins are intentional.
- **JOIN type** — INNER when LEFT was needed drops rows silently; verify cardinality assumptions.
- **Function on indexed column in WHERE** — `UPPER(col) = ...` prevents index use; restructure or use generated columns.
- **Overly nested queries** — flatten 3+ levels of nesting into named CTEs.
- **Dead code** — remove commented-out blocks, unused CTEs, columns selected but never referenced downstream.

### Python checklist (`.py` files)

Before editing: read `/Users/patrickbyrne/styleguides/conventions/python.md` — source of truth for conventions. Grep the repo for existing utilities/helpers/clients the draft may have reinvented.

Do not rewrite architecture, move logic between layers, add tests, or change public signatures (unless a signature is objectively bloated: unused params, `Optional` that's never `None`).

Fix directly:
- Imports — remove unused, consolidate duplicates. Import modules not objects unless the object import significantly improves readability.
- Error handling — no bare `except`, no `except Exception: pass`, no try/except that catches and re-raises unchanged. Use custom domain exceptions where the caller needs to branch on error type.
- Verbosity — collapse `x = foo(); return x` → `return foo()`. Collapse obvious multi-step builds into comprehensions. Remove intermediate vars used once.
- Over-tiered helpers — a helper encoding multiple length/size thresholds often collapses to one simpler rule; check.
- Misleading names — flag function names that encode type assumptions (`_alpha_`, `_str_`) when the function does no such validation.
- Reuse — if a repo helper exists, use it.
- Dead code — remove commented-out blocks, unused locals, unreachable branches.
- Guard clauses — early returns over deep nesting.

Flag, do not silently fix — list every instance in the closing summary:
- External dependencies called directly (S3, databases, APIs, filesystem) without dependency injection — add a `# TODO: inject <dependency> as parameter` comment, don't refactor architecture.
- `mock.patch` / `unittest.mock.Mock` / `MagicMock` in tests — convention is fakes, not mocks; don't remove without a replacement fake.
- No-op test assertions — `assert True`, `assert result is not None`, `assert mock.called` with no check of *what* it was called with.
- Clear-box test assertions — asserting on internal calls (`.assert_called_once_with`) rather than observable outcomes.

### Comments checklist (all in-scope files)

Work only on lines actually added or changed; don't rewrite pre-existing comments the author didn't touch unless directly adjacent to changed code and now wrong. Read each file before editing so you judge against the real code, not just the diff hunk.

Make every surviving comment earn its place:
- **Why, not what.** Delete comments that restate the code. Keep only non-obvious why — a decision, constraint, workaround, gotcha.
- **Don't comment around unreadable code.** If a line needs prose to follow, that's a naming/extraction problem — flag it, don't paper over it.
- **Ask for the why you don't have.** If a comment should explain a why you can't infer from code/diff/git history, use AskUserQuestion (batch, one call, up to four) rather than inventing a rationale or deleting it as noise. If unanswered, leave it and flag it.
- **Docstrings: what, then why. Nothing else.** One plain sentence on what, then the why. Cut the surrounding-world tour (caller usage, pipeline position, upstream/downstream). Strip `Args:`/`Returns:` blocks that only re-type the signature; keep a param line only if it carries a why the type can't.
- **Direct.** Cut hedging/preamble (`# Note that...`, `# Here we...`, `# For now,`). Lead with the point.
- **No dangling doc references.** Remove/rewrite pointers to docs, tickets, or scratch files (`MIGRATION_PLAN.md`, etc.) not tracked in git and not landing in this change. Inline the one needed fact instead of the pointer if genuinely load-bearing. A stable ticket ID for *why* is fine; "explained elsewhere" is not.
- **No names.** Strip `# per Sarah`-style attribution. Keep the substance, drop the attributor — git blame has the rest.
- **No plan or phase references.** Delete anything anchored to delivery narrative (`# phase 2 cleanup`, `# temporary until the migration lands`). State the condition instead of the schedule if the future change is real and load-bearing.
- **Write it to still be true in two years.** Soften anything pinned to drift-prone details (line numbers, other functions' names, "as of <date>", "currently we..."). State the rule, not today's number — unless the number is a real invariant.
- **Unverbose.** Collapse banner/box comments and redundant docstring boilerplate. Delete commented-out code and generator `# TODO`s the author didn't mean.

Flag, don't silently fix:
- A comment that contradicts the code it sits on.
- A "why" comment you suspect is incorrect.
- Code that's only legible because of its comment — name the file/line, say what to extract or rename.
- A why you asked about and didn't get an answer to.

Don't add new comments unless removing one leaves genuinely unexplained behavior, and only when it's a why you actually know.

### Output

End with: `Hardened N files (X sql, Y python, Z comment-only). Applied A changes. Flags (if any): ...`
