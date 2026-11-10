---
name: review-pr
description: Review a pull request before merge with three independent reviewers (/code-review, /security-review and /sec-context:review), each in a fresh agent, then show the findings and record them on the PR as comments. Use when asked to review a PR, to run "the reviews" or "the merge reviews", or before merging any PR.
argument-hint: "<PR number> [code-review level, default high]"
---

# Review a pull request

The user invoked this with: `$ARGUMENTS`

The first argument is the PR number (required; if it's missing, use the PR for the current branch from `gh pr view --json number`, and stop if there is none). An optional second argument is the `/code-review` effort level; default `high`. Never use `max` or `ultra` unless the user asked for it.

## 1. Look at the PR

Run `gh pr view <N> --json number,title,body,headRefName,headRefOid,baseRefName,isDraft,files,url` and `gh pr diff <N> --name-only`.

- **Docs-only PR**: every changed file is documentation (Markdown, `docs/`, an OpenSpec archive or spec sync, comments only). It gets the code review but skips both security reviews. Say so.
- Read the project's CLAUDE.md for its own review rules (security-sensitive areas to look at closely, a different code-review level, extra reviewers). The project's rules win where they differ from this skill.
- If `/sec-context:review` isn't available (the sec-context plugin isn't enabled here), say so and continue with the other two; don't install it without asking.

## 2. Run the reviewers in fresh agents

Each reviewer runs in its **own new agent**, never in the session that wrote the change, so it reviews with fresh eyes. Start them in parallel in one message. Every brief says:

- Review PR #<N> (`<headRefOid>`) only. Read the change with `gh pr diff <N>` and files with `git show <headRefOid>:<path>` (after `git fetch origin pull/<N>/head` if needed).
- **Never check out, switch, reset or stash in the main checkout**, and make no edits, commits, pushes or PR comments. Return the findings as text.
- If a command is denied, stop and report it; don't work around it.
- Return each finding with file:line, severity, what's wrong, a concrete failure scenario and a suggested fix; or say plainly that there were no findings. Include what was reviewed (head commit, files).

The three reviewers:

1. **Code review**: an agent that invokes the `code-review` skill with args `<level> <N>`.
2. **Security review** (skip for docs-only): the `security-review` skill reviews *the current branch*, not a PR number, so this agent must run in an isolated worktree (`isolation: "worktree"`), where it may run `gh pr checkout <N>` *inside that worktree only*, confirm `git rev-parse HEAD` equals `<headRefOid>`, then invoke `security-review`. If it can't confirm the head commit, it stops and reports rather than reviewing the wrong branch.
3. **sec-context review** (skip for docs-only): an agent that invokes the `sec-context:review` skill with args `<N>`.

## 3. Check and show the findings

Before showing anything, read each finding against the code yourself: drop ones that are plainly wrong (cite why), and mark ones that describe code the PR didn't change as **pre-existing**. Then show the user one list per reviewer, ranked by severity, with your verdict on each (real / not real / pre-existing). Count them; don't estimate.

## 4. Record them on the PR

Post one comment per reviewer with `gh pr comment <N> --body-file <file>` (write the body to a temp file, not inline), headed with the reviewer, the head commit reviewed and the date, including "No findings" when there were none. Invoking this skill is the user's request to post these comments, so they need no separate approval once the user has seen the findings.

Exception: if the repository is **public** (`gh repo view --json visibility`) and a finding is an exploitable security issue that isn't fixed yet, don't publish its details. Post that a security finding is being fixed in this PR, fix it, and give the details once the fix is pushed, or ask the user.

## 5. After the reviews

- For each real finding, either fix it on the PR branch or record in the PR why it's deferred; deferring needs the user's agreement. Offer to fix; don't start without it, and don't commit or push without approval.
- Re-run the affected reviewer(s) on the new head after fixes that touch security-sensitive code, and post the re-review the same way.
- When fixes change what the PR does, update the PR title and description too: squash-merging makes them the commit.
- Don't merge. Say the PR is reviewed and wait for the user's go-ahead (then merge or enable squash auto-merge, whichever the project uses).
