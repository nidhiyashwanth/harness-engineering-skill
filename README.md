# harness-engineering skill

A Claude skill that helps you build and improve the **harness** around a coding agent — the
instructions, tools, environment, state, and feedback (everything outside the model's weights) that
turn a capable model into one that reliably finishes real tasks in a repository.

It distills the [Learn Harness Engineering](https://walkinglabs.github.io/learn-harness-engineering/)
course (12 lectures) into an actionable, router-style skill. Fittingly, the skill *itself* follows
the course's principles: a short `SKILL.md` entry file that routes to focused topic references
revealed on demand (Lecture 04), rather than one giant instruction file.

## What it does

- **Diagnoses** a failing agent by attributing the symptom to a harness subsystem, then repairing
  that layer (instead of reaching for a different model).
- **Sets up** an agent-ready repo from scratch: routing `AGENTS.md`, executable verification + a
  Definition of Done, an initialization phase, `PROGRESS.md` / `DECISIONS.md`, a feature list,
  completion gates, and observability.

## Layout

```
harness-engineering/
├── SKILL.md                 # router: 5 subsystems, diagnostic table, build workflow
├── references/              # read on demand, one per topic (mapped to lectures)
│   ├── foundations.md
│   ├── repository.md
│   ├── instruction-files.md
│   ├── continuity.md
│   ├── scope-and-feature-lists.md
│   ├── verification.md
│   └── observability.md
├── assets/                  # templates to copy into a target repo
│   ├── AGENTS.md.template
│   ├── PROGRESS.md.template
│   ├── DECISIONS.md.template
│   └── feature-list.template.json
└── evals/evals.json         # test prompts (+ drafted assertions) for the skill-creator eval loop
```

## Install

### Claude Code plugin marketplace

Add this repository as a marketplace, then install the plugin:

```text
/plugin marketplace add nidhiyashwanth/harness-engineering-skill
/plugin install harness-engineering@harness-engineering-skill
```

The plugin installs the self-contained skill from
`plugins/harness-engineering/skills/harness-engineering/`, including `references/`, `assets/`, and
`evals/`.

### Codex plugin marketplace

Add this repository as a marketplace, then install `harness-engineering` from the Codex plugin
directory:

```bash
codex plugin marketplace add nidhiyashwanth/harness-engineering-skill
```

Codex reads `.agents/plugins/marketplace.json`, which points at the same self-contained plugin
payload under `plugins/harness-engineering/`.

### Direct skill install

You can also copy this repository root as a plain skill folder:

- Claude Code personal skill: `~/.claude/skills/harness-engineering/`
- Codex personal skill: `~/.agents/skills/harness-engineering/`

When copying manually, copy the whole directory, not just `SKILL.md`, so `references/` and `assets/`
are available for progressive disclosure.

## Credit

Content distilled from *Learn Harness Engineering* by Walking Labs. This repo restates and
reorganizes that material as an operational skill; see the course for the full lectures.
