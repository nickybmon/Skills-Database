---
name: review-pr
description: Review a contributor's pull request thoroughly and post inline comments as a pending (unsubmitted) GitHub review. Use when asked to review a PR, draft review comments, or follow up on a PR that has had review rounds already. Usage: /review-pr <PR_NUMBER>
---

# Review PR

Review a contributor's pull request and post inline comments as a **pending review**, so the user can inspect everything in GitHub before the contributor sees it.

## Setup

Find the repo with `gh repo view --json nameWithOwner -q .nameWithOwner` (or use the repo the user names). Use it as `<owner>/<repo>` below.

## What this does

1. Fetches the PR diff and reads all changed files in full.
2. Sorts issues into three severity levels:
   - **Critical / bugs**: broken or incorrect behavior.
   - **Concerns**: design choices, edge cases, or patterns to address before merge.
   - **Minor**: small suggestions, style, non-blocking improvements.
3. Drafts inline comments in a warm, direct tone. Acknowledge the intent of the work, then be clear and specific about the issue.
4. Shows the drafts to the user for approval.
5. Posts the approved comments as a single pending review.
6. Does NOT submit the review or write a summary. The user does both in GitHub.

## Tone for inline comments

- Lead with what the contributor was trying to do or what works.
- Then state the concern directly. Do not soften the technical facts.
- For bugs, name what will go wrong and suggest the fix.
- For concerns, explain the risk and offer a concrete alternative.
- For minor items, keep them brief and mark them non-blocking.
- Write like a thoughtful collaborator, not a checklist.

## Process

1. `gh pr view <PR_NUMBER> --repo <owner>/<repo>` for metadata and the head SHA.
2. `gh pr diff <PR_NUMBER> --repo <owner>/<repo>` for the full diff.
3. Fetch the PR's version of each changed file through the GitHub API to get exact line numbers.
4. Read each file fully for context, not just the changed lines.
5. Check for an existing pending review from you on this PR. If one exists, tell the user before continuing.
6. Draft all inline comments and show them to the user. Wait for approval.
7. Post as one pending review:
   `gh api repos/<owner>/<repo>/pulls/<PR_NUMBER>/reviews --method POST --input <json_file>`
   Leave out the `event` field so the review stays pending.
8. Confirm the posted comment count matches the draft.
9. Tell the user to open GitHub, Files changed, to inspect and submit in their own words.

## What NOT to do

- Do not submit the review.
- Do not draft or post an overall summary.
- Do not post anything before the user approves the drafts.
- Do not blur severity levels. Be clear what is a bug and what is a suggestion.

## After one or two unresolved review rounds

If a PR has been through one or two comment cycles and still has issues, recommend fixup-and-squash-merge instead of another comment round. Ask the user before doing any of it.

1. Check `maintainer_can_modify`: `gh api repos/<owner>/<repo>/pulls/<N> --jq '.maintainer_can_modify'`
2. Add the contributor's fork as a local remote and check out their branch.
3. Push fixup commits to their branch that resolve the issues.
4. Squash merge, preferably through the GitHub web UI. It injects the correct `Co-authored-by` trailers from the PR history, so the contributor gets credit. The CLI needs the email supplied by hand:
   `gh pr merge <N> --repo <owner>/<repo> --squash --subject "..." --body "Co-authored-by: <name> <<id>+<username>@users.noreply.github.com>"`
5. Comment on the closed PR. Explain each change briefly and thank the contributor, so they see what good looks like without feeling blocked or lectured.

Only merge when the user asks for the merge in that same request.
