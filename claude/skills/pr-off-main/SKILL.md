---
name: pr-off-main
description: "Open a PR for the current branch against main, then chain into pr-desc (title/description) and post-pr (Slack draft). Use when Patrick asks to open/create/ship a PR from scratch, as opposed to pr-desc (existing PR only) or post-pr (existing PR only)."
---

# PR off main

End-to-end PR creation: push the branch, open the PR, write the description, draft the Slack post. This is the "start to finish" version — `pr-desc` and `post-pr` on their own only operate on a PR that already exists.

## Step 1 — Sanity checks

1. Confirm we're inside a git repo. If currently on `main`/`master`, this is the common case, not an error — don't stop and ask. Instead: `git pull` to bring the base branch up to date, then create and check out a new feature branch from it (`git checkout -b <name>`). Derive `<name>` as a short kebab-case slug from the pending changes (uncommitted diff, or recent conversation context) — no need to confirm the branch name first, creating a local branch is reversible. If genuinely nothing to name it from (no uncommitted changes, no clear task context), ask briefly rather than guessing a generic name.
2. Determine the base branch: prefer `main`, fall back to `master` if that's what the remote's default branch actually is (check `git remote show origin` or `gh repo view --json defaultBranchRef` if unsure — don't assume).
3. Check for commits ahead of base: `git log <base>..HEAD --oneline`. If empty, stop and tell Patrick there's nothing to PR yet.
4. Check `git status` for uncommitted changes. If there are any, surface them and ask whether to leave them uncommitted (fine, they just won't be in the PR) or stop so he can commit first — don't commit on his behalf.
5. Check whether a PR already exists for this branch (`gh pr view --json number 2>/dev/null`). If one does, skip straight to Step 3 (pr-desc) — don't create a duplicate.

## Step 2 — Push and open the PR

1. Show Patrick the branch, base, and commit list from Step 1, and confirm before doing anything visible to others (push + PR creation is shared state, not local/reversible).
2. Push the branch: `git push -u origin HEAD` if it has no upstream yet, otherwise a plain `git push`.
3. Create the PR with a throwaway placeholder — it gets overwritten in Step 3, so don't spend effort on it: `gh pr create --base <base> --fill`. Use `--fill` so it seeds from the commit log rather than prompting.

## Step 3 — Write the real title/description

Invoke the `pr-desc` skill. It will pull the diff, draft a title + body, and may ask Patrick clarifying questions before updating the PR — let it run its own flow, don't shortcut it.

## Step 4 — Draft the Slack announcement

Once `pr-desc` has finished updating the PR, invoke the `post-pr` skill. It checks CI/review health first and will stop and report back if something's failing or unresolved, rather than drafting the Slack message — respect that stop, don't push past it.

## Notes

- Don't merge, don't mark ready-for-review if opened as draft, and don't send the Slack message — this skill only gets a PR to the point where Patrick can review and send it himself, same as `post-pr` on its own.
- If any step fails (push rejected, `gh pr create` errors, no CI checks configured yet), stop and report rather than guessing a workaround.
