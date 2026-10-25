---
name: adversarial-review
description: Run a hard-nosed, read-only adversarial review of the current branch's diff or a specific PR. Use to stress-test a branch before opening or merging a PR, sanity-check work in progress, or review a PR before merge. Triggers: "adversarial review", "review this branch", "run the gate", "sanity-check this PR".
---

# Adversarial Review

Review a diff as a skeptical senior engineer whose job is to prove the change is wrong. Passing tests and a clean build do not mean the change is correct. They mean nothing tested the failure. This review is read-only: no commits, no pushes, no edits to PR text, no comments on GitHub.

## Invocation

```
adversarial review [PR_NUMBER]
```

`PR_NUMBER` is optional. With it, review that PR. Without it, review the current branch.

## Steps

### 1. Get the diff

Find the base branch (usually `main`; check `git symbolic-ref refs/remotes/origin/HEAD` if unsure).

- **With a PR number:** `gh pr checkout <PR_NUMBER>`, then `git diff <base>...HEAD`.
- **Without:** `git diff <base>...HEAD`.

If the diff is empty, report "no changes, review skipped" and stop.

### 2. Build symbol context

List every function, class, type, or exported name the diff adds, changes, or removes. For each, search the repo for callers and callees (`grep -rn "<symbol>"`, limited to the project's source file types). Cap output at about 50 lines per symbol. This surfaces integration risk that the diff alone hides.

### 3. Read the project rules

Read `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, or similar files at the repo root, if they exist. Any hard rule in them (branch policy, required headers, version bumps, forbidden patterns) is a gate in step 4.

### 4. Review every gate

Do not skip a gate, even if it looks fine.

**Correctness (red)**
- Logic errors, off-by-one, wrong conditions, unhandled empty or null input.
- Race conditions, shared mutable state, callbacks assumed to run on a specific thread.
- Resource lifecycle: leaks, retain cycles, listeners or handles never released.

**Regression risk (red)**
- Does the change affect any code path outside its stated scope?
- Check every call site from step 2. Does the new behavior break any caller?
- Does the change reopen a bug a prior commit fixed? (`git log -p -- <file>`)
- Is any existing test, invariant, or documented contract now violated?

**Error handling (red)**
- Are all failure paths handled and logged with enough context to diagnose?
- Can a failure leave shared state half-initialized?

**Performance (yellow)**
- Work inside hot paths (loops, per-frame, per-request) that belongs in setup.
- New blocking calls on latency-sensitive threads.

**Project rules (yellow)**
- Every hard rule found in step 3.
- No debug artifacts, hardcoded local paths, or secrets.

**Test coverage (yellow)**
- Mutation-test in your head: if this hunk is reverted, does any test fail? If not, say so.
- Does the PR describe how to test the change manually?

### 5. Report

Be terse. Cite `file:line` for every finding. Give each finding a check that can prove it wrong.

```
## Adversarial Review: <branch or PR>

### BLOCK (must fix before merge)
- file:42 | Claim: <how the change is wrong>
  Evidence: <what in the code supports it>
  Check: <command, test, or code reading that would disprove it>

### NOTE (should fix, or explicitly accept the risk)
- file:17 | <concise description>

### Clean
- <gate name>: no issues found. Checked: <what you looked at>
```

**Severity**
- `BLOCK`: a crash, data loss, silent misbehavior, or a hard project-rule violation.
- `NOTE`: real risk, not merge-blocking. The maintainer should acknowledge it.

If a gate has no findings, mark it Clean and name what you checked. A review with no findings and no named checks means you did not review.
