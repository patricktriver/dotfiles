Run the following linting steps using the Bash tool, in order:

1. Run `uv run ruff format` and capture the output.
2. Before fixing, capture the pre-fix dirty state: `git diff --name-only`
3. Run `uv run ruff check --select I --fix` to auto-fix import ordering.
4. After fixing, capture the post-fix dirty state: `git diff --name-only`
5. Run `uv run mypy --no-incremental --pretty` and capture the output.

After all steps:
- Compute which files ruff actually changed: files present in the post-fix dirty list that weren't already dirty before the fix (i.e. new entries added by ruff). Also include files that were already dirty but whose diff changed due to ruff (use `git diff --name-only` before and after to compare).
- More precisely: run `git diff --name-only` after the fix and compare to before. Any file whose diff changed is a ruff-modified file.
- If ruff modified any files, stage only those specific files (`git add <file1> <file2> ...`).
- Do NOT use `git add -u` or `git add .` — only add the files ruff touched.
- Before committing, inspect the staged diff (`git diff --cached`) and classify the changes:
  - **Formatting/import-only** (whitespace, line breaks, import reordering, quote style): commit automatically with the message `linting`.
  - **Any logic change** (renamed variables, reordered expressions, removed/added statements, changed return values): do NOT commit. Show the user the diff and ask for confirmation before proceeding.
- Report a summary of what ruff and mypy found/fixed.
