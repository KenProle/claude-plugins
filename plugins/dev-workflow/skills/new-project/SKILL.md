---
name: new-project
description: Set up a new or existing repository with Ken's development practices - PR-and-review workflow, dependency policy, OpenSpec, the kenprole-plugins marketplace, a daily vulnerability check and GitHub merge settings - adapted to the project's language and stack.
argument-hint: "[all | claude-md | plugins | openspec | vulncheck | github]"
disable-model-invocation: true
---

# Set up a project

The user invoked this with: `$ARGUMENTS` (empty means `all`).

This copies practices into the repository so they bind everyone who works on it, including cloud sessions and other contributors. Personal habits live in the user-level CLAUDE.md and are **not** copied here.

## 0. Survey first, change nothing

Find out and show the user, briefly:

- Language(s), package managers and their manifest and lock files, supported runtime versions, how the project is built, tested and distributed (binaries, container image, library, web app), and its license (`LICENSE`; if none, ask, don't pick one).
- Whether a CLAUDE.md, `.claude/settings.json`, `openspec/`, `.github/workflows/` or a README already exist, and what they say.
- The git remote, default branch, and repository visibility and plan (`gh repo view --json nameWithOwner,visibility,defaultBranchRef,isInOrganization`).
- Whether the working tree is clean and the default branch matches its remote.

Then ask, in one `AskUserQuestion` (multi-select), which pieces to set up, preselecting what `$ARGUMENTS` named: CLAUDE.md sections, plugins in `.claude/settings.json`, OpenSpec, vulnerability-check workflow, GitHub merge settings and ruleset.

## 1. Work on a branch

Unless the repository has no commits yet, create a branch (`setup-dev-workflow`) from an up-to-date default branch. Everything below lands in one PR, reviewed with `/dev-workflow:review-pr` like any other. Ask before each commit and push.

## 2. CLAUDE.md sections (`claude-md`)

- If there is no CLAUDE.md, write one first the way `/init` would (what the project is, commands, architecture), from the survey; keep it to facts an agent can't easily derive.
- Add the sections from `templates/claude-md-sections.md` (in this skill's folder), filled in for this project: every `{{…}}` replaced with real values (`{{PLUGIN_VERSION}}` is the `version` in this plugin's `plugin.json`, `{{DATE}}` is today), and parts that don't apply removed rather than left vague. Use the stack's real tools: for example `go mod graph` / `govulncheck` / pkg.go.dev for Go, `npm ls --all` / `npm audit` / npmjs.com for npm, `pip install` into a scratch venv and `pip show` / `pip-audit` / PyPI for Python. Check that any tool you name exists for this stack; don't name one from memory without checking it.
- If CLAUDE.md already has rules on any of these topics, merge them: keep the project's stricter or more specific rules, and show the user each conflict instead of silently replacing it.
- Fill "Security-sensitive areas" from the actual code (files that run subprocesses, build paths from input, handle credentials, parse untrusted input). If there's nothing yet, say "None yet; add areas here as they appear."

## 3. Plugins (`plugins`)

Create or merge `.claude/settings.json` (keep every existing key):

```json
{
  "extraKnownMarketplaces": {
    "kenprole-plugins": {
      "source": { "source": "github", "repo": "KenProle/claude-plugins" }
    }
  },
  "enabledPlugins": {
    "dev-workflow@kenprole-plugins": true,
    "sec-context@kenprole-plugins": true
  }
}
```

Add `"trim-reaper@kenprole-plugins": true` only if the user wants it (ask; it suits projects with real dependency trees). Make sure `.claude/settings.local.json` is gitignored and `.claude/settings.json` is not.

## 4. OpenSpec (`openspec`)

- Look up the latest OpenSpec release live (npm `@fission-ai/openspec`, and its GitHub releases); never use a version from memory. Pin that exact version in CLAUDE.md, and follow the dependency policy when choosing it (recent-release scrutiny, advisories).
- Check `openspec init --help` for the current options, then run it for Claude Code in the repository root. Turn telemetry off if it isn't already (`openspec config set telemetry.enabled false`).
- Replace the generated `openspec/config.yaml` with `templates/openspec-config.yaml`, filled in: the project name, a one-line description, and only constraints the code can't tell an agent (who the users are, what must never be claimed or done, external services allowed).
- Add `openspec validate --all --strict` to CI if CI exists.

## 5. Vulnerability check workflow (`vulncheck`)

If the project has third-party dependencies, add `.github/workflows/vulncheck.yml` with one job named `Vulnerability check` that runs the stack's advisory checker on pull requests, pushes to the default branch, a daily schedule (pick an odd minute, not `:00`) and manual dispatch, with `permissions: contents: read`.

- Go: `go run golang.org/x/vuln/cmd/govulncheck@<pinned version> ./...` after `actions/setup-go` with `go-version-file: go.mod`. npm: `npm audit --audit-level=high` from the directory with `package-lock.json`. Python: `pip-audit` against the pinned requirements. Other stacks: find the established checker and confirm with the user.
- Pin every action and tool to the latest release, checked live; comment each pin with "Pinned; upgrade deliberately".
- Add `.github/dependabot.yml` for `github-actions` only, grouped, monthly, so action pins don't go stale.
- Note in CLAUDE.md that GitHub disables a schedule after 60 days without repository activity.

## 6. GitHub settings (`github`)

These change the repository on GitHub, so show the exact settings and ask before each `gh api` call.

- Merge settings: squash merge only (title and body from the PR), delete head branches on merge, allow auto-merge.
- A ruleset on the default branch requiring a pull request and the CI jobs by their exact names. Rulesets aren't enforced on private repositories on GitHub Free; if that's the case, say so instead of creating one that does nothing. Leave the repo owner an admin bypass for emergencies, and record the ruleset and its required checks in CLAUDE.md.

## 7. Finish

Open the PR, then run `/dev-workflow:review-pr <N>` (the CLAUDE.md and workflow changes are not docs-only). Summarize what was set up, what was skipped and why, and anything the user must still do themselves (for example enabling GitHub Pages, adding secrets, or installing the plugins in other clones).
