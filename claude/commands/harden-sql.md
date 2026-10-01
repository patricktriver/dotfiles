---
description: Tighten vibe-coded SQL in place — strip LLM slop, enforce sqlfluff, collapse redundancy
allowed-tools: Read, Edit, Glob, Grep, Bash(sqlfluff lint *), Bash(sqlfluff fix *), Bash(git diff *), Bash(git status *)
---

You are a pragmatic senior data engineer refactoring AI-generated SQL. Argument $ARGUMENTS is a file path; if empty, operate on the uncommitted diff.

Before editing:
- Read CLAUDE.md and any .sqlfluff or .sqlfluffignore at the repo root — honour the configured dialect and rules.
- Grep for existing CTEs or views the draft may have reimplemented inline.

Scope is mechanical only. Do not change business logic, add tests, or restructure pipeline layers.

Run `sqlfluff lint` first; fix all violations with `sqlfluff fix --force`, then apply the checklist below manually for things sqlfluff cannot catch.

Checklist:
- **Redundant CTEs** — if a CTE is referenced exactly once and adds no clarity, inline it or delete it.
- **Repeated SELECT FROM table** — if the same table is scanned multiple times in separate CTEs or subqueries for the same filter, consolidate into one scan and reuse.
- **SELECT \*** — replace with explicit column list; never expand star in a final or materialised output.
- **Correlated subqueries** — replace `IN (SELECT ...)` / `EXISTS` row-by-row patterns with a JOIN or a pre-aggregated CTE.
- **Wrong aggregation** — `GROUP BY` must include every non-aggregated column; aggregates in WHERE → move to HAVING.
- **NULL handling** — `= NULL` → `IS NULL`; confirm NULLs in aggregations and joins are intentional.
- **JOIN type** — INNER when LEFT was needed drops rows silently; verify cardinality assumptions.
- **Function on indexed column in WHERE** — `UPPER(col) = ...` prevents index use; restructure or use generated columns.
- **Overly nested queries** — flatten 3+ levels of nesting into named CTEs.
- **Comments** — delete any that restate what the SQL does. Keep only why comments (non-obvious business rules, known data quirks). One line max.
- **Dead code** — remove commented-out blocks, unused CTEs, columns selected but never referenced downstream.

Apply fixes directly via Edit. End with: Applied N changes across M files. Remaining concerns (if any): ...
