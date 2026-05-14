# Agent Skills

Personal Codex and coding-agent skills synchronized across machines.

These are opinionated personal skills distilled from my substantial hands-on experience building, reviewing, and maintaining Java/Kotlin systems.

## Layout

- `skills/<skill-name>/SKILL.md` - individual skill instructions.
- `skills/<skill-name>/scripts/` - optional helper scripts.
- `skills/<skill-name>/references/` - optional reference material loaded on demand.
- `skills/<skill-name>/assets/` - optional reusable assets or templates.

## Install

### Codex

In Codex, ask `$skill-installer` to install:

```text
https://github.com/nerzhulart/agent-skills/tree/main/skills/kotlin-java-code-guides
```

Restart Codex if the skill does not appear automatically.

To use this repo as a live local source instead of installing a copy:

```bash
git clone git@github.com:nerzhulart/agent-skills.git ~/work/agent-skills
mkdir -p ~/.agents
ln -s ~/work/agent-skills/skills ~/.agents/skills
```

If `~/.agents/skills` already exists, move or merge it before creating the symlink.

### Claude Code

In Claude Code, add this repository as a plugin marketplace and install the bundled skills plugin:

```text
/plugin marketplace add nerzhulart/agent-skills
/plugin install agent-skills@agent-skills
```

Restart Claude Code after installing the plugin.

To use this repo as a live local source instead of installing a plugin:

```bash
git clone git@github.com:nerzhulart/agent-skills.git ~/work/agent-skills
mkdir -p ~/.claude/skills
ln -s ~/work/agent-skills/skills/kotlin-java-code-guides ~/.claude/skills/kotlin-java-code-guides
```

Restart Claude Code after creating or updating skill links.
