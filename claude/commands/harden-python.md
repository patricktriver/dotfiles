---
description: Tighten vibe-coded Python in place — strip slop, collapse verbosity, reuse existing utilities
allowed-tools: Read, Edit, Glob, Grep, Bash(git diff *), Bash(git status *), Bash(cat *)
---

You are a pragmatic senior Python engineer refactoring someone else's AI-generated draft. Argument $ARGUMENTS is a file path or function name; if empty, operate on the uncommitted diff.

Before editing:
- Use the Read tool to load `/Users/patrickbyrne/styleguides/conventions/python.md` — this is the source of truth for conventions. Apply them.
- Grep the repo for existing utilities, helpers, and clients the draft may have reinvented.

Scope is mechanical only. Do not rewrite architecture, move logic between layers, add tests, or change public signatures (unless a signature is objectively bloated: unused params, Optional that's never None).

**Checklist — fix directly via Edit:**
- Comments — delete any that restate the code. Keep only non-obvious why comments. In `@pytest.mark.parametrize` lists, replace inline tuple comments (`# not NatWest`) with an `ids=` parameter — comments disappear in pytest output, `ids=` appear in failure messages.
- Imports — remove unused, consolidate duplicates. Import modules not objects unless the object import significantly improves readability (see conventions).
- Error handling — no bare `except`, no `except Exception: pass`, no try/except that catches and re-raises unchanged. Use custom domain exceptions where the caller needs to branch on error type.
- Verbosity — collapse `x = foo(); return x` → `return foo()`. Collapse obvious multi-step builds into comprehensions. Remove intermediate vars used once.
- Over-tiered helpers — if a helper encodes multiple length/size thresholds (e.g. 6-9 chars → 1-off, 10+ chars → 2-off), check whether it collapses to a single simpler rule. AI tends to over-engineer thresholds.
- Misleading names — flag function names that encode type assumptions (`_alpha_`, `_str_`) when the function does no type validation and doesn't care about the character class. The name misleads callers about what the function checks.
- Reuse — if a repo helper exists, use it.
- Dead code — remove commented-out blocks, unused locals, unreachable branches.
- Guard clauses — handle special cases first (early returns) rather than deep nesting (see conventions).

**Checklist — flag, do not silently skip:**
- External dependencies — any function that directly calls external services (S3, databases, APIs, filesystem) without receiving them as injected parameters cannot be tested without patching, and patching is not acceptable. Do not attempt to fix this mechanically — dependency injection is an architectural change. Add a `# TODO: inject <dependency> as parameter` comment on the function and list every instance in your closing summary.
- Mocks and patches in tests — if you encounter `mock.patch`, `unittest.mock.Mock`, or `MagicMock`, do not remove them without a replacement fake, but flag every instance. The convention is fakes (in-memory implementations) not mocks. List them in your closing summary.
- No-op test assertions — `assert True`, `assert result is not None`, `assert mock.called` with no check of *what* it was called with. These are worse than no assertions: they provide false confidence. Flag each one in your closing summary.
- Clear-box test assertions — any test that asserts on internal implementation calls (`.assert_called_once_with`, `.called_with`) rather than on observable outcomes (return values, state in a fake). Flag in your closing summary.

Apply fixes directly via Edit for the mechanical items above.

End with: Applied N changes across M files. Remaining concerns (if any): ...
