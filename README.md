# AI Coding Skills

Reusable planning skills for Claude and Codex. Each skill uses the shared Agent
Skills format, so there is one source file for both platforms and no
platform-specific copies or installer scripts.

## Skills

- **bugfix-plan** - Plan targeted fixes with dependency tracing, root-cause
  analysis, and regression coverage.
- **migration-plan** - Plan feature migrations with dependency and parity
  tracking.
- **new-feature-plan** - Plan features using existing structure, conventions,
  and integration points.
- **planning-subagents** - Delegate bounded planning evidence collection to
  smaller workers while the primary model owns judgment and synthesis.
- **refactor-plan** - Plan refactors with dependency, cycle, boundary, and
  maintainability analysis.

## Structure

Every skill follows the same minimal structure:

```text
skills/
├── bugfix-plan/
│   └── SKILL.md
├── migration-plan/
│   └── SKILL.md
├── new-feature-plan/
│   └── SKILL.md
├── planning-subagents/
│   └── SKILL.md
└── refactor-plan/
    └── SKILL.md
```

Skill directory names use kebab case. Each `SKILL.md` starts with the portable
frontmatter required by Claude and Codex:

```yaml
---
name: skill-name
description: Explain what the skill does and when it should be used.
---
```

## Use with Claude

Copy a skill directory into one of Claude Code's skill locations:

- Personal: `~/.claude/skills/<skill-name>/`
- Project: `.claude/skills/<skill-name>/`

Invoke a skill with its slash command, such as `/bugfix-plan`, or let Claude
select it from the description.

Claude.ai can also discover this repository layout when the repository is
connected through Customize → Skills.

## Use with Codex

Copy a skill directory into one of Codex's skill locations:

- Personal: `~/.codex/skills/<skill-name>/`
- Project: `.codex/skills/<skill-name>/`

Invoke a skill by mentioning it with `$`, such as `$bugfix-plan`, or let Codex
select it from the description.

## Prerequisite

These skills use [assistgraph](https://github.com/assistagent/assistgraph) for
dependency and declaration analysis. Install the current CLI before using a
planning skill:

```bash
npm install -g assistgraph
```

The skills query `.assist/graph/graph.json` through compact commands such as
`assistgraph files`, `symbols`, `file`, `deps`, `dependents`, `path`,
`communities`, `cycles`, `orphans`, and `status`. They do not load the complete
JSON graph or require MCP.
