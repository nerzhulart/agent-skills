# Agent Skills

Personal Codex and coding-agent skills synchronized across machines.

## Layout

- `skills/<skill-name>/SKILL.md` - individual skill instructions.
- `skills/<skill-name>/scripts/` - optional helper scripts.
- `skills/<skill-name>/references/` - optional reference material loaded on demand.
- `skills/<skill-name>/assets/` - optional reusable assets or templates.

## Install

```bash
git clone git@github.com:nerzhulart/agent-skills.git ~/work/agent-skills
mkdir -p ~/.agents
ln -s ~/work/agent-skills/skills ~/.agents/skills
```

If `~/.agents/skills` already exists, move or merge it before creating the symlink.
