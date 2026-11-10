<!-- Template for /dev-workflow:new-project. Fill every {{…}} for the project and
     delete what doesn't apply; never leave a placeholder in a project's CLAUDE.md.
     Bullets marked (if …) are kept only when that condition holds. -->

## Pull requests and reviews

<!-- Synced from KenProle/claude-plugins dev-workflow {{PLUGIN_VERSION}} template on {{DATE}}. Compare with the template before changing these rules. -->

- Changes go through pull requests, so each one is documented: work on a branch, open a PR whose description says what changed and why (link the OpenSpec change if there is one), and let CI pass. Never commit straight to `{{DEFAULT_BRANCH}}`.
- Before merging, run `/dev-workflow:review-pr <N>`. It runs `/code-review`, `/security-review` and `/sec-context:review` on the PR in fresh agents (not the session that wrote the change) and records what each found, or that it found nothing, as PR comments. Fix each real finding or note in the PR why it's deferred; deferring needs the repo owner's agreement. Re-review after fixes that touch the security-sensitive areas below. A docs-only PR can skip both security reviews but not the code review.
- The plugins come from the `kenprole-plugins` marketplace, which `.claude/settings.json` enables for this project; install them once per clone with `claude plugin install dev-workflow@kenprole-plugins --scope project` (and the same for `sec-context`{{TRIM_REAPER_INSTALL}}).
- Merge with squash so `{{DEFAULT_BRANCH}}` keeps one commit per change. The PR title and body become the commit, so write the title like the existing history and update both when review fixes change what the PR does. Bring a branch up to date by merging `{{DEFAULT_BRANCH}}` into it, not by rebasing and force-pushing.
- (if a ruleset is set up) `{{DEFAULT_BRANCH}}` is protected by a ruleset that requires a pull request and the CI jobs named {{REQUIRED_CHECKS}}. Renaming a job means updating the ruleset too.

Before writing or changing any code that falls under `/sec-context:guide`'s description, load it for that topic first. This applies everywhere in the project, not only in the areas below. Its description is the full list; common cases are parsing untrusted input, subprocesses, file paths, credentials and tokens, HTML, endpoints, headers and dependencies. Without the plugin, say so and carry on.

Security-sensitive areas to review closely:
{{SECURITY_SENSITIVE_AREAS}}
<!-- e.g. code that runs subprocesses or builds shell commands; file paths built from input;
     authentication, sessions and secrets; parsing of untrusted input; network calls to
     user-chosen hosts; install scripts that run with elevated rights. Name the real files. -->

## Third-party dependencies

<!-- Synced from KenProle/claude-plugins dev-workflow {{PLUGIN_VERSION}} template on {{DATE}}. -->

This is the project's one dependency policy{{OPENSPEC_POLICY_POINTER}}. It also applies to the other third-party pieces the build uses: {{OTHER_THIRD_PARTY}}.

Where dependencies are declared, and how they are pinned: {{DEPENDENCY_FILES}}.

- Minimize third-party dependencies to reduce ongoing updates and maintenance. Prefer {{STDLIB}}, platform APIs, or what the project already has when they meet the requirements.
- Add a library only when its benefits justify its ongoing maintenance, security and upgrade costs, and record that justification ({{JUSTIFICATION_PLACE}}). For simple, narrowly scoped functionality, consider a small local implementation when it is easier to maintain and verify. For security-sensitive functionality, especially cryptography, authentication, and complex parsing, use or retain established libraries or vetted platform APIs rather than hand-rolling replacements. Retain other specialized libraries when replacement would increase security, correctness, or maintenance risk.
- Inspect the resolved transitive dependency tree before adding or upgrading a library ({{TREE_COMMAND}}). Prefer suitable options with fewer transitive dependencies, and avoid bringing in large dependency trees for functionality the project barely uses. Explain why any substantial increase is necessary and assess its update burden.
- When changing existing dependency usage, identify unused or redundant libraries and unnecessary optional integrations as removal candidates. Remove them only within the authorized task scope and after checking code, dynamic loading, configuration, scripts, tests and build tooling ({{BUILD_TOOLING}}). Preserve behavior and supported runtime compatibility ({{RUNTIME_SUPPORT}}), verify the changes, and explain justified retention. For a full reduction pass, use `/trim-reaper:trim-reaper assess`.
- Before adding or upgrading a library, verify the exact package name and publisher against the project's own documentation or repository, and check its registry entry live ({{REGISTRIES}}). Do not trust registry metadata alone. Never select package names, versions or commit hashes from model memory. Investigate lookalike names, unexpected ownership changes, and unfamiliar new packages before proceeding.
- Use the latest stable, supported release compatible with the project. Scrutinize very recent releases and unexpected maintainer or release changes; absence of an advisory is not evidence that a release is safe. Explain any choice of an older release.
- The project is {{LICENSE}}-licensed and is distributed as {{DISTRIBUTION}}. Before adding or upgrading a library, check its license and those of its transitive dependencies for compatibility with that distribution. Preserve required notices and flag unclear or incompatible terms for the repo owner before proceeding.
- Assess maintenance through support status, unresolved issues, security responsiveness, and compatibility with supported runtimes. Treat release frequency as one signal; a mature, stable library may need few releases. Reject archived, deprecated, or abandoned projects.
- Check current security advisories from official project sources and relevant advisory databases ({{ADVISORY_SOURCES}}). Report the sources and date checked, affected versions, and applicability to this project. Avoid versions with known applicable vulnerabilities. If advisory lookup is unavailable, stop and report that limitation before adding or upgrading the dependency; do not describe an unchecked version as vulnerability-free.
- Keep updates scoped to the task. Let the package manager resolve transitive dependencies; avoid unrelated upgrades, forced overrides, and manual edits to lockfiles that bypass compatibility constraints. Prefer a compatible parent-library update when a transitive dependency needs remediation.
- Before updating, review release notes, migration guidance, and dependency constraints for breaking changes. Explain concrete effects on this project and the required migration work. If an update requires a broader migration or hits a compatibility blocker, stop and report the evidence and options before expanding scope. Do not repeatedly attempt speculative upgrades or fixes; resume only with new evidence or an agreed migration plan.
- If you cannot verify these requirements, stop and report the blocker. Do not add or upgrade the dependency unless the repo owner explicitly accepts the identified risk.

## Spec-driven development (OpenSpec)

<!-- Keep only if the project uses OpenSpec. -->

Features are specified before they're built, with [OpenSpec](https://github.com/Fission-AI/OpenSpec) {{OPENSPEC_VERSION}} (pinned; upgrade deliberately with `openspec update`).

- `openspec/specs/` holds the living specs (current behavior, per capability); `openspec/changes/<name>/` holds in-flight changes: `proposal.md`, delta `specs/`, `design.md`, `tasks.md`. Archiving a change merges its deltas into the living specs.
- `openspec/config.yaml` holds project-wide constraints and per-artifact rules that are injected into every artifact; read it before writing one.
- Use the `/opsx:explore`, `/opsx:propose`, `/opsx:apply` and `/opsx:archive` skills, or the CLI: `openspec status --change <name>`, `openspec validate --all --strict`.
- Requirements and scenarios carry stable IDs in their headers (`### Requirement: AB-3 …`, `#### Scenario: AB-3.1 …`), one prefix per capability. OpenSpec matches requirements by header text, so never renumber or reuse an ID. Tests start their names with the scenario ID they cover.
- Capability prefixes in use: none yet.
