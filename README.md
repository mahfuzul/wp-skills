# wp-updates

An [Agent Skill](https://agentskills.io) that runs the routine WordPress update cycle for any git-tracked WordPress project, using WP-CLI:

1. Pulls the base branch (`master` or `main`) and creates a dated branch, e.g. `wp_updates_nov_26`.
2. Updates WordPress core (`wp core update`) and commits it.
3. Updates all plugins (`wp plugin update --all`), re-checks for leftovers (usually license-gated pro plugins), and commits.
4. Reports theme updates and updates a theme only if you approve, with its own commit.
5. Merges the branch into the `stage`/`staging` branch for local review.
6. Pushes only after you say so.

Core, plugins and themes each get their own commit, so any one can be reverted alone. Nothing is pushed without your explicit approval.

## Prerequisites

- [WP-CLI](https://wp-cli.org) (`wp`) with working database access to the site
- `git`, with a base branch (`master`/`main`) and a `stage`/`staging` branch
- An AI coding agent that supports Agent Skills

## Install

### Any agent (Claude Code, Codex, OpenCode, and others)

```sh
npx skills add <your-github-user>/wp-updates-skill
```

### Claude Code plugin marketplace

```
/plugin marketplace add <your-github-user>/wp-updates-skill
/plugin install wp-updates@wp-updates-skill
```

### Manual

Copy `skills/wp-updates/` into your agent's skills folder:

| Agent | Global | Per project |
|---|---|---|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Codex | `~/.codex/skills/` | `.agents/skills/` |
| OpenCode | `~/.config/opencode/skill/` | `.opencode/skill/` |
| Others (e.g. Command Code) | see the agent's docs | many read `.agents/skills/` |

Skill folder locations vary by agent and version, so check your agent's documentation if the skill isn't picked up.

## Use

Ask your agent, for example:

> Run the monthly WordPress updates.

or invoke it directly, e.g. `/wp-updates` in Claude Code. Run it from inside the WordPress project.

## License

MIT
