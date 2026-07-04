# Harness Engineering Plugin

This plugin packages the `harness-engineering` Agent Skill for Claude Code and Codex.

The skill helps coding agents make repositories more reliable by improving the harness around the
model: instructions, tools, environment, state, and feedback. It is meant for tasks such as making a
repo agent-ready, writing or refactoring `AGENTS.md` / `CLAUDE.md`, defining verification commands,
adding continuity files like `PROGRESS.md` and `DECISIONS.md`, and diagnosing why an agent keeps
stalling or declaring work done without evidence.

## Install

Claude Code:

```text
/plugin marketplace add nidhiyashwanth/harness-engineering-skill
/plugin install harness-engineering@harness-engineering-skill
```

Codex:

```bash
codex plugin marketplace add nidhiyashwanth/harness-engineering-skill
```

## Contents

- `skills/harness-engineering/SKILL.md` - the router workflow and trigger description.
- `skills/harness-engineering/references/` - detailed topic references loaded on demand.
- `skills/harness-engineering/assets/` - templates for target repositories.
- `skills/harness-engineering/evals/` - evaluation prompts for skill iteration.

## Usage Examples

After installing, ask Claude Code or Codex for tasks like:

- `Use harness-engineering to make this repo agent-ready.`
- `Use harness-engineering to diagnose why the agent keeps saying done before the app works.`
- `Use harness-engineering to create AGENTS.md, verification commands, and a Definition of Done.`

## License

MIT. See the repository `LICENSE` file.
