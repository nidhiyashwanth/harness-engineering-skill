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

Copy the `harness-engineering/` directory into your skills directory (e.g.
`~/.claude/skills/harness-engineering/`), or package it with the skill-creator's `package_skill.py`
and install the resulting `.skill` file.

## Credit

Content distilled from *Learn Harness Engineering* by Walking Labs. This repo restates and
reorganizes that material as an operational skill; see the course for the full lectures.
