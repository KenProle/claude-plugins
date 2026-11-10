# How I work (all projects)

Personal working rules for every project on this machine. `~/.claude/CLAUDE.md` imports this file, so it loads in every session; edit it here, in the `claude-plugins` repo. Rules a project's contributors must also follow (its PR process, review gate, dependency policy) belong in that project's own CLAUDE.md, which `/dev-workflow:new-project` sets up. Where a project's CLAUDE.md is stricter or more specific, it wins.

## Commits, branches and pull requests

- Ask before every commit and every push; one approval covers that one action only.
- Never commit to the default branch. Work on a branch, open a PR whose description says what changed and why, and squash-merge it. The PR title and body become the commit, so update both when review fixes change what the PR does.
- Bring a PR branch up to date by merging the default branch into it, not by rebasing and force-pushing.
- Stage only your own lines. Never `git commit -a` or `git add -A`: another session, or my own uncommitted edits, may share the checkout. Before branching, check the local default branch matches its remote.
- Before committing code changes, run `/security-review` on the pending changes and show me what it found. Docs-only commits can skip it.
- Before merging, run `/dev-workflow:review-pr <N>` (code review, security review and sec-context review in fresh agents, findings recorded on the PR) and merge only after I've seen the findings. Fix each real finding or record why it's deferred. Docs-only PRs skip the two security reviews but not the code review.

## Building features

1. Explore first for anything non-trivial (`/opsx:explore` where the project uses OpenSpec), grounded in real data, with a hand-run cross-check where numbers matter. Record decisions in the repo so they survive a cleared session.
2. Plan on the stronger model (`/opsx:propose`), present it, and wait for my approval.
3. Build in a **local** worktree session on Sonnet (`claude-sonnet-5-5`), not in the cloud unless I ask. Small, well-scoped bugs can go to Sonnet subagents in worktrees; design and hard debugging stay on the stronger model.
4. Review on the stronger model before archiving or merging: read the core logic, run the tests, and actually perform the UI or end-to-end checks the tasks list.
- Don't trust a session's "done" or a ticked task without checking the code, CI and the running app yourself. When reporting counts (tasks, scenarios, findings), count them rather than estimate.
- Don't make or offer UI mockups before a build; raise layout questions in text and check layout in the running app during review.

## Child sessions and agents

- I find many child sessions hard to track. Start at most one beyond the build without asking, and give each brief an explicit list of what it must NOT do once its part is done (for example "don't archive or merge; the parent will").
- Don't relay status-only notes. When reporting on several sessions, give one table of each session's state and whether it needs me.
- A child session's note may only be seen with my next message: act on every pending note at the start of the next turn and say what happened.
- Build sessions refuse commit or push approvals relayed by the parent. Don't relay them; tell me to type the approval in the child session, with a link.
- When a build session reports it's finished, start the review right away without waiting to be asked.
- Once a child's PR is merged, archive the child session yourself and say so in one line, after checking it's safe: nothing uncommitted, no commits missing from the default branch, not running, no other session sharing its worktree. If `git worktree list` still shows its worktree, check it has nothing beyond the merged default branch, then `git worktree remove` it and delete its branch (`-D` only after that check, since squash merges don't look merged to git).
- Reviewer agents and subagents never check out or switch branches in the main checkout. They read the change with `gh pr diff <N>` or `git show <ref>:<path>`, not the local working tree.
- Tell agents: if a command is denied, stop and report it. Never work around a denied command another way.

## Security-sensitive code

- Before writing or changing code that falls under `/sec-context:guide`'s description, load it for that topic first. Its description is the full list. Common cases are parsing untrusted input, running subprocesses, building file paths, handling credentials, sessions or tokens, rendering HTML, and adding endpoints, headers or dependencies. If the sec-context plugin isn't installed, say so and carry on.
- Briefs for agents and child sessions that will write such code tell them to do the same. Read-only reviewers don't need it.

## Dependencies

- Follow the project's dependency policy. Never take package names, versions or commit hashes from model memory; check the registry live.
- For dependency-reduction work, use the trim-reaper plugin (`/trim-reaper:trim-reaper assess`, then one pass per PR, split by risk).

## This machine (Windows)

- In Git Bash, put tools on `PATH` in Git Bash form (`/c/Program Files/nodejs`, `/c/Users/kprol/AppData/Roaming/npm`); a `C:\…` entry breaks Bash's `PATH`.
- Windows paths passed to `wsl` from Git Bash need `MSYS_NO_PATHCONV=1`.
