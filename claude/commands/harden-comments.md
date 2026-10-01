---
description: Harden the comments in files touched by the PR — direct, why-focused, no dangling doc refs, not brittle, not verbose
allowed-tools: Read, Edit, Glob, Grep, AskUserQuestion, Bash(git diff *), Bash(git log *), Bash(git status *), Bash(gh pr view *), Bash(gh pr diff *)
---

!`gh pr diff --name-only 2>/dev/null || git diff main...HEAD --name-only`

You are a pragmatic senior engineer cleaning up comments in someone's PR. Argument $ARGUMENTS may be a file path to scope to; if empty, operate on every file in the PR listed above. If `gh` returned no PR, fall back to the uncommitted/branch diff.

Scope is the comments only — code, signatures, and logic stay untouched. Work only on lines the PR actually added or changed (use the diff); don't rewrite pre-existing comments the author didn't touch unless they're directly adjacent to changed code and now wrong.

Read each target file before editing so you judge the comment against the real code, not the diff hunk alone.

**Make every surviving comment earn its place. Fix directly via Edit:**

- **Why, not what. The code speaks for itself.** Delete comments that restate the code (`# increment counter`, `// loop over users`). Keep a comment only when it explains *why* — a non-obvious decision, a constraint, a workaround, a gotcha the next reader would trip on. If a comment can't be rephrased as a "why", it's noise.
- **Don't comment around unreadable code.** If a line genuinely needs prose to be followable, that's a code smell, not a comment gap. Flag it (`this needs a comment because the code is opaque — consider naming/extracting instead`) rather than papering over it. Leave the code alone; surfacing it is the job.
- **Ask for the why you don't have.** When a comment clearly *should* explain a why and you can't infer it from the code, diff, or git history, don't invent a plausible-sounding rationale and don't delete the comment as noise. Use AskUserQuestion to ask what the real reason is (batch questions — one call, up to four). If unanswered, leave the comment and flag it.
- **Docstrings: what, then why. Nothing else.** One plain sentence saying what the function does, then the why — the reason it exists, the constraint it satisfies, the gotcha in its inputs. That's the whole shape. Cut the surrounding-world tour: how the caller uses it, what runs before or after it, where it sits in the pipeline, what the wider system is doing. Include external context only when the function's own code or inputs are unreadable without it (an input arrives pre-filtered, a value is already normalised upstream, an argument means something non-obvious) — and then just the one fact, not the narrative. Strip `Args:`/`Returns:` blocks that only re-type the signature; keep a param line when it carries a why the type can't (units, allowed values, ownership).
- **Direct.** Cut hedging and preamble — `# Note that...`, `# Here we...`, `# This is basically just...`, `# For now,`. Lead with the point. One clause beats three.
- **No dangling doc references.** Remove or rewrite comments that point to docs, tickets, design pages, or READMEs that aren't in this PR and likely never will be (`# see the design doc`, `# per the spec`, `# refer to confluence`). Treat scratch docs generated during the session as never existing — `MIGRATION_PLAN.md`, `ARCHITECTURE.md`, `NOTES.md` and friends that aren't tracked in git are not getting committed, so any comment referencing them is dead on arrival. Check with git before trusting a path. If the referenced thing is genuinely load-bearing, inline the one fact the reader needs instead of the pointer. A stable ticket/issue ID for *why a thing exists* is fine to keep; a pointer that just says "explained elsewhere" is not.
- **No names.** Strip references to individual people — `# per Sarah`, `# Dave asked for this`, `# as discussed with the platform team lead`. Whoever it is will move teams and the comment becomes an unanswerable question. Keep the substance of the decision, drop the attributor. Git blame already records who.
- **No plan or phase references.** Delete anything anchored to a delivery narrative the code can't see: `# stage 5 will introduce X`, `# phase 2 cleanup`, `# temporary until the migration lands`, `# part of the Q3 rollout`. Nobody reading this later knows what stage 5 was. If the future change is real and load-bearing, state the condition instead of the schedule (`# single-tenant only; needs a tenant key before multi-tenant use`). Otherwise cut it.
- **Write it to still be true in two years.** Every surviving comment must survive refactors, reorgs, and staff turnover. Soften anything pinned to details that drift: exact line numbers, function names of *other* code, precise counts/values that aren't invariants, "as of <date>", "currently we...", "the new approach", "recently changed". Restate at the level that stays true — the rule, not the current number; the constraint, not today's implementation of it. If a specific value is a real invariant, keep it and say why it's fixed.
- **Unverbose.** Collapse multi-line banner/box comments and redundant docstring boilerplate to a single line where the detail adds nothing. Cut comments that duplicate a clear type hint or a self-documenting name. Delete commented-out code and `# TODO` left by the generator that the author didn't mean.

**Flag, don't silently fix:**
- A comment that contradicts the code it sits on — the code or the comment is wrong; surface it, don't guess.
- A "why" comment you suspect is incorrect — flag rather than delete; the author may know something you don't.
- Code that's only legible because of its comment — name the file and line and say what should be extracted or renamed.
- A why you asked about and didn't get an answer to.

Don't add new comments unless removing one leaves genuinely unexplained behavior — and when you do add one, it's a why you actually know, not a guess.

End with: Hardened comments in N files (M deleted, K rewritten). Flags (if any): ...
