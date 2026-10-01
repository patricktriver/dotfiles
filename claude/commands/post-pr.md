---
description: Draft a Slack #dev announcement for a PR (current branch or specified PR)
---

Post a draft Slack message announcing a PR to the #dev channel.

PR target: $ARGUMENTS

Steps:
1. Determine which PR to use:
   - If $ARGUMENTS is empty, run `gh pr view --json number,title,url,body` to detect the PR for the current branch.
   - If $ARGUMENTS contains a PR number (e.g. `#123` or `123`) or a URL, run `gh pr view <that> --json number,title,url,body` instead.
   - If no PR is found, tell the user and stop.
2. Check PR health before doing anything else:
   a. Run `gh pr view <number> --json statusCheckRollup` (or `gh pr checks <number>`) to check CI/CD status (lint, tests, build, etc). Note any check that is failing or erroring — pending checks are fine to mention but not blocking.
   b. Run `gh pr diff <number> --name-only` and check whether every changed file is under `snowflake/schemachange/views/`. This mirrors the repo's "Auto approve" workflow, which auto-approves views-only PRs. If every file matches, the PR is already auto-approved and needs no visibility push: tell Patrick it's views-only and already auto-approved, then stop — skip the Slack draft entirely. If any file falls outside that folder, the auto-approve workflow won't be green — continue drafting the Slack message as normal.
   c. Run `gh pr view <number> --json reviewThreads,comments` to find unresolved review threads and PR-level comments, especially from review bots (Copilot, CodeRabbit/coderabbitai). Flag any unresolved thread or bot comment that reads as a requested change (not just a nit acknowledged or already resolved).
   d. If there are failing checks and/or unresolved/unaddressed bot comments, stop and summarize them clearly for Patrick (check name + link, or comment body + link) instead of drafting the Slack message. Ask whether he wants to fix them first or proceed anyway.
   e. Only continue to step 3 once CI is green and there's nothing outstanding — or Patrick explicitly says to proceed regardless.
3. Look at the PR diff (`gh pr diff <number>`) and body to understand what changed and its rough complexity (very simple / small / moderate / large — based on lines changed and number of files, not a rigid rubric).
4. Compose the Slack message in exactly this format:

```
:github: {PR Title} (<{link to PR}|#{PRNumber}>)

_{one very short sentence: what the PR does, plus a quick note on complexity}_
```

   Use Slack's `<url|text>` mrkdwn syntax so the PR number is a clickable hyperlink — never paste the raw URL as trailing plain text, it won't get embedded.

   Example:
   ```
   :github: Add submit-invoice-legacy endpoint (<https://github.com/org/repo/pull/174|#174>)

   _Copies code over and points integrations frontend at this one for now — very simple, not many lines._
   ```

   Keep the description sentence terse and factual, matching the tone of the example. Don't editorialize or pad it.

5. Resolve the #dev channel (use slack_search_channels if needed to confirm the channel ID).
6. Use slack_send_message_draft (NOT slack_send_message) to stage the message in #dev — never send directly, this is always a draft for Patrick to review and send himself.
7. Show Patrick the exact message text you drafted so he can see it without switching to Slack.
