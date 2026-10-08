# AI Agent Skills

A shared collection of [Agent Skills](https://skills.sh) for AI coding agents such as Claude Code, Codex, Gemini CLI, Cursor, GitHub Copilot and others.

A skill is a folder with a `SKILL.md` file: a set of instructions, plus any scripts or reference files it needs, that an agent loads when a task calls for it. Skills in this repo are installed with the [`skills`](https://github.com/vercel-labs/skills) CLI (`npx skills`).

## Requirements

- Node.js 18 or later (for `npx`)
- At least one supported AI agent installed

## Quick start

Install skills from this repo. The CLI asks which skills and which agents you want:

```bash
npx skills add juzmatch/ai-agent-management-public
```

See which skills are available before installing:

```bash
npx skills add juzmatch/ai-agent-management-public --list
```

## Installing

### Pick specific skills

```bash
npx skills add juzmatch/ai-agent-management-public --skill <skill-name>

# several at once
npx skills add juzmatch/ai-agent-management-public --skill <skill-a> <skill-b>
```

### Pick specific agents

```bash
# one agent
npx skills add juzmatch/ai-agent-management-public --agent claude-code

# several agents
npx skills add juzmatch/ai-agent-management-public --agent claude-code codex gemini-cli

# every supported agent
npx skills add juzmatch/ai-agent-management-public --agent '*'
```

### Project or global

| Scope | Flag | Installs to | Use it when |
| --- | --- | --- | --- |
| Project (default) | none | the agent's folder in the current project, e.g. `.claude/skills/` | the skill belongs to one project and should be committed for the team |
| Global | `-g` | the agent's folder in your home directory, e.g. `~/.claude/skills/` | you want the skill in every project |

```bash
npx skills add juzmatch/ai-agent-management-public -g
```

### Install everything without prompts

Useful for scripts and CI:

```bash
# all skills, all agents, no prompts
npx skills add juzmatch/ai-agent-management-public --all

# all skills, chosen agents, globally, no prompts
npx skills add juzmatch/ai-agent-management-public --skill '*' --agent claude-code codex gemini-cli -g -y
```

### Copy instead of symlink

By default the CLI keeps one copy of each skill and symlinks it into each agent's folder. If your setup does not support symlinks (some Windows setups, Docker volumes), use `--copy`:

```bash
npx skills add juzmatch/ai-agent-management-public --copy
```

## Supported agents

Pass these values to `--agent`. The paths show where each agent reads skills from.

| Agent | `--agent` | Project path | Global path |
| --- | --- | --- | --- |
| Claude Code | `claude-code` | `.claude/skills/` | `~/.claude/skills/` |
| OpenAI Codex | `codex` | `.agents/skills/` | `~/.codex/skills/` |
| Gemini CLI | `gemini-cli` | `.agents/skills/` | `~/.gemini/skills/` |
| Antigravity | `antigravity` | `.agents/skills/` | `~/.gemini/antigravity/skills/` |
| Cursor | `cursor` | `.agents/skills/` | `~/.cursor/skills/` |
| GitHub Copilot | `github-copilot` | `.agents/skills/` | `~/.copilot/skills/` |
| Windsurf | `windsurf` | `.windsurf/skills/` | `~/.codeium/windsurf/skills/` |
| OpenCode | `opencode` | `.agents/skills/` | `~/.config/opencode/skills/` |
| Kiro CLI | `kiro-cli` | `.kiro/skills/` | `~/.kiro/skills/` |
| Qwen Code | `qwen-code` | `.qwen/skills/` | `~/.qwen/skills/` |
| Cline | `cline` | `.agents/skills/` | `~/.agents/skills/` |
| Roo Code | `roo` | `.roo/skills/` | `~/.roo/skills/` |
| Amp | `amp` | `.agents/skills/` | `~/.config/agents/skills/` |
| Goose | `goose` | `.goose/skills/` | `~/.config/goose/skills/` |

The CLI supports more agents than this. For the full, current list, see the [skills CLI README](https://github.com/vercel-labs/skills#supported-agents).

## Managing installed skills

```bash
npx skills list                 # show installed skills (add -g for global)
npx skills update               # update all installed skills to the latest version
npx skills update <skill-name>  # update one skill
npx skills remove <skill-name>  # remove one skill (add -g for global)
```

## Repository layout

```
.
├── skills/
│   └── <skill-name>/
│       ├── SKILL.md        # required: frontmatter + instructions
│       ├── scripts/        # optional: helper scripts the skill runs
│       ├── references/     # optional: docs the agent reads on demand
│       └── assets/         # optional: templates, images, other files
└── template/
    └── SKILL.template.md   # starting point for a new skill
```

The CLI finds every `skills/<skill-name>/SKILL.md` in this repo. The template is named `SKILL.template.md` so it is never installed.

## Adding a skill

1. Copy the template into a new folder. The folder name should match the skill's `name`:

   ```bash
   mkdir -p skills/my-skill
   cp template/SKILL.template.md skills/my-skill/SKILL.md
   ```

2. Edit `skills/my-skill/SKILL.md`. The frontmatter needs two fields:

   ```markdown
   ---
   name: my-skill
   description: What the skill does and when the agent should use it.
   ---
   ```

   - `name`: lowercase letters, numbers and hyphens; unique in this repo.
   - `description`: agents read this to decide when to load the skill, so say what it does and what should trigger it.
   - Optional: set `metadata.internal: true` to hide a work-in-progress skill. It is then only installable with `INSTALL_INTERNAL_SKILLS=1`.

3. Test it locally before pushing:

   ```bash
   npx skills add . --list                              # is it discovered?
   npx skills add . --skill my-skill --agent claude-code # try it in an agent
   ```

4. Commit and push. Anyone can then install it with `npx skills add juzmatch/ai-agent-management-public --skill my-skill`.

## Security

This is a public repository. Do not put secrets, API keys, internal URLs, or customer data in any skill. Skills run with the permissions of the agent that loads them, so review a skill's `SKILL.md` and scripts before installing it.

## License

[MIT](LICENSE)
