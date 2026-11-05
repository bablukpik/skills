# Skills

Reusable agent skills. Each skill is a directory with a `SKILL.md` file that tells an agent how to do a specific job.

Skills follow the [Agent Skills](https://agentskills.io/) format and install with the [skills CLI](https://skills.sh/docs/cli).

## Available skills

| Skill | What it does |
| --- | --- |
| [generate-api](generate-api/SKILL.md) | Design, implement, refactor, and review production-ready REST APIs. |

See [generate-api/README.md](generate-api/README.md) for what that skill covers and when to use it.

## Install

Always install from GitHub, including while working in this repository. Push to `main` first, then install. The CLI installs for every agent it detects, including Cursor, Claude Code, OpenCode, and Codex:

```bash
npx skills add bablukpik/skills
```

Full steps for project versus global scope and choosing agents are in [docs/installing.md](docs/installing.md).

## Repository layout

```text
skills/
├── README.md
├── docs/
│   └── installing.md
└── generate-api/
    ├── README.md
    └── SKILL.md
```

Add another skill as its own directory next to `generate-api/`:

```bash
npx skills init my-skill
```

Then edit `my-skill/SKILL.md`: set `name` and `description` in the frontmatter, and write the instructions in the body. `name` must match the directory name.
