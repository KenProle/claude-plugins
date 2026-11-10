# claude-plugins

Ken Prole's Claude Code plugin marketplace (`kenprole-plugins`) and personal working rules, kept in one place so new projects and machines can reuse them.

| Plugin | What it does | Source |
|---|---|---|
| `dev-workflow` | `/dev-workflow:review-pr <N>` reviews a pull request with `/code-review`, `/security-review` and `/sec-context:review` in fresh agents and records the findings on the PR. `/dev-workflow:new-project` sets up a repository with the PR-and-review workflow, dependency policy, OpenSpec, these plugins, a daily vulnerability check and GitHub merge settings, adapted to its stack. | [`plugins/dev-workflow`](plugins/dev-workflow) |
| `sec-context` | Security review and secure-coding guidance against the sec-context AI code security anti-patterns. | [KenProle/sec-context](https://github.com/KenProle/sec-context) |
| `trim-reaper` | Dependency-reduction passes that preserve behavior, security and compatibility. | [KenProle/trim-reaper](https://github.com/KenProle/trim-reaper) |

## Use in a project

Add to the project's `.claude/settings.json` (or run `/dev-workflow:new-project`, which does it for you):

```json
{
  "extraKnownMarketplaces": {
    "kenprole-plugins": { "source": { "source": "github", "repo": "KenProle/claude-plugins" } }
  },
  "enabledPlugins": {
    "dev-workflow@kenprole-plugins": true,
    "sec-context@kenprole-plugins": true
  }
}
```

Or install for yourself everywhere:

```bash
claude plugin marketplace add KenProle/claude-plugins
claude plugin install dev-workflow@kenprole-plugins
claude plugin install sec-context@kenprole-plugins
```

`user/CLAUDE.md` expects `sec-context` (its review and guide skills), so install both.

## Personal rules

[`user/CLAUDE.md`](user/CLAUDE.md) holds working rules that apply to every project on my machine. It is loaded through an import in `~/.claude/CLAUDE.md`:

```
@C:/Work/Code/claude-plugins/user/CLAUDE.md
```

Rules a project's other contributors must follow go in that project's own CLAUDE.md instead; `new-project` copies them from [`templates/claude-md-sections.md`](plugins/dev-workflow/skills/new-project/templates/claude-md-sections.md) and marks the copy with the template version and date.

## License

MIT. `sec-context` and `trim-reaper` are separate repositories with their own licenses.
