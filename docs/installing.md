# Installing skills

Install from GitHub: [github.com/bablukpik/skills](https://github.com/bablukpik/skills). That includes the person editing this repository. Push changes to `main`, then install or update from `bablukpik/skills`.

The `skills` CLI installs a skill for the agents it finds on the machine. The same `SKILL.md` works in Cursor, Claude Code, OpenCode, and Codex. Run the command from the application you want the skill to affect, unless you pass `-g`.

## Agents

Leave `--agent` off and the CLI installs for every agent it detects. Pass `-y` to skip the confirmation prompt:

```bash
npx skills add bablukpik/skills --skill generate-api -y
```

Name the agents when you want this exact set:

```bash
npx skills add bablukpik/skills --skill generate-api --agent cursor claude-code opencode codex -y
```

`--agent '*'` installs for every detected agent. Agent names are `cursor`, `claude-code`, `opencode`, and `codex`.

Inside a project, Cursor, OpenCode, and Codex read `.agents/skills/`. The CLI keeps one copy there. Claude Code reads `.claude/skills/`, so the CLI adds a symlink from `.claude/skills/generate-api` back to that copy. Updating the copy updates all four.

A global install (`-g`) uses a different directory per agent:

| Agent | `--agent` | Project | Global |
| --- | --- | --- | --- |
| Cursor | `cursor` | `.agents/skills/` | `~/.cursor/skills/` |
| Claude Code | `claude-code` | `.claude/skills/` | `~/.claude/skills/` |
| OpenCode | `opencode` | `.agents/skills/` | `~/.config/opencode/skills/` |
| Codex | `codex` | `.agents/skills/` | `~/.agents/skills/` |

## Install into the current project

Project scope is the default when the current directory is a project.

```bash
cd /path/to/your-app
npx skills add bablukpik/skills --skill generate-api -y
```

Omit `--skill` to install every skill in the repo.

The same GitHub source works as a URL:

```bash
npx skills add https://github.com/bablukpik/skills --skill generate-api -y
```

Or the skill file on `main`:

```bash
npx skills add https://github.com/bablukpik/skills/blob/main/generate-api/SKILL.md -y
```

## Install for every project

Global scope needs `-g`. The CLI still installs for each detected agent, using that agent's global directory from the table above.

```bash
npx skills add bablukpik/skills --skill generate-api -g -y
```

To limit a global install to Cursor, Claude Code, OpenCode, and Codex:

```bash
npx skills add bablukpik/skills --skill generate-api --agent cursor claude-code opencode codex -g -y
```

## Check and remove

```bash
npx skills list
npx skills list -g
```

```bash
npx skills remove generate-api -y
npx skills remove generate-api -g -y
```

## After the skill changes

Push to `main` on [github.com/bablukpik/skills](https://github.com/bablukpik/skills), then update the installed copy from that remote:

```bash
npx skills update generate-api
npx skills update generate-api -g
```

`skills update` follows the GitHub remote recorded at install time. Install with `npx skills add bablukpik/skills` so later updates come from this repository.

## Publishing

Push this repository. Anyone, including the author, installs with:

```bash
npx skills add bablukpik/skills
```

There is no separate registry submission. [skills.sh](https://skills.sh/) can list a skill after people install it from the public GitHub repo.
