# Agent Skills

Personal Codex and coding-agent skills synchronized across machines.

These are opinionated personal skills distilled from my substantial hands-on experience building, reviewing, and maintaining Java/Kotlin systems.

## Available Skills

- `kotlin-java-code-guides`: general non-Android JVM Kotlin/Java design, implementation, coroutines, Flow, and review guidance.
- `intellij-platform-kotlin-guides`: IntelliJ Platform-specific Kotlin/Java lifecycle, coroutine scope, `Disposer`, listener, and editor/viewer guidance.

## Install

### Codex

In Codex, ask `$skill-installer` to install one or both skills:

```text
https://github.com/nerzhulart/agent-skills/tree/main/skills/kotlin-java-code-guides
https://github.com/nerzhulart/agent-skills/tree/main/skills/intellij-platform-kotlin-guides
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
ln -s ~/work/agent-skills/skills/intellij-platform-kotlin-guides ~/.claude/skills/intellij-platform-kotlin-guides
```

Restart Claude Code after creating or updating skill links.
