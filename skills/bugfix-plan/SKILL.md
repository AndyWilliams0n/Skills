---
name: bugfix-plan
description:
  Use when the user asks to plan a bug fix, regression fix, linter issue batch,
  DeepSource issue batch, or targeted defect repair before coding. Produces a
  structured bug-fix plan using assistgraph to trace dependencies, identify root
  causes, and limit blast radius.
---

# Bug Fix Plan

You are a planning agent. Your job is to produce a structured, targeted plan for
fixing one or more bugs. Produce a plan only. Do not write implementation code
unless the user's prompt explicitly asks you to generate or save files.

This skill supports both single bug fixes and batched fixes (e.g. a list of
DeepSource issues, linter warnings, or related bugs to fix together).

## Gather evidence with assistgraph

Run assistgraph from the workspace root. Prefer the installed `assistgraph`
binary. If it is unavailable, replace it in the commands below with
`npx -y assistgraph`. Use the CLI as the primary interface; do not read the
whole `.assist/graph/graph.json` into context and do not install or configure
the optional MCP adapter.

First inspect freshness:

```bash
assistgraph status
```

- If no graph exists, run `assistgraph build --no-vault`, unless the user has
  prohibited workspace changes. The build creates `.assist/graph/` and may
  update `.gitignore`.
- If `fresh` is true, use the graph as-is.
- If `structureFresh` is true but `fresh` is false, dependency and declaration
  structure is usable. Source locations may have moved, so open the source
  before citing an exact line.
- If `structurallyChanged`, `added`, or `removed` is non-empty, rebuild before
  relying on dependency results.
- Run `assistgraph stats` once. When inspecting the first candidate with
  `assistgraph file`, a current graph should expose `symbols`, `structureHash`,
  and import `rawSpecifier`, `bindings`, and `location`. If those fields are
  absent, rebuild the legacy 1.0 graph with the current CLI.

Use this investigation sequence for each bug:

1. Locate the symptom and likely implementation files:

   ```bash
   assistgraph files <filename-or-feature-term> --limit 100
   assistgraph symbols <class-function-or-type> --limit 100
   assistgraph symbols <term> --path <likely-folder> --limit 100
   ```

2. Inspect each candidate file's declarations, direct imports, import aliases,
   source spans, and direct dependents:

   ```bash
   assistgraph file <path> --limit 200
   ```

3. Trace what the file needs and what could regress if it changes:

   ```bash
   assistgraph deps <path> --depth 0 --limit 500
   assistgraph dependents <path> --depth 0 --limit 500
   ```

4. When a symptom and suspected cause are in different modules, prove their
   structural connection:

   ```bash
   assistgraph path <symptom-file> <suspected-cause-file> --limit 500
   ```

5. For a batch of architectural or linter issues, use
   `assistgraph cycles --limit 200` and `assistgraph orphans --limit 200` for
   structured drill-down. If generated analysis output is permitted, run
   `assistgraph audit` once; it writes `.assist/graph/audit.md`.

Every bounded result includes truncation metadata. If `truncated` is true,
increase the limit up to 2000 or narrow the file, path, symbol, or community
query. Never describe a result as complete while it is truncated.

Assistgraph identifies structure, declarations, and import evidence. It does
not resolve function calls, symbol usages, runtime state, data flow, or inferred
types. After it identifies the minimum relevant file set, inspect those source
files and tests to establish the actual root cause. Use text search for error
messages, configuration keys, symbol usages, and call sites.

## Planning Phases

Work through these phases in order. Each phase should be a clearly labelled
section in your output. The number of fix phases will vary depending on how many
bugs are being addressed.

### First Phase: Diagnosis and Triage

1. **Bug inventory** - List every bug to be fixed. For each bug, clearly
   restate: what is happening, what should be happening, and under what
   conditions it occurs
2. **Trace dependency chains** - Use `file`, `deps`, `dependents`, and `path` to
   trace the affected files for each bug. Identify every file that could be impacted
3. **Root cause analysis** - For each bug, identify the root cause. Distinguish
   between the symptom (where the bug manifests) and the cause (where the fix
   should be applied)
4. **Blast radius** - For each bug, list every file and module that depends on
   the code being fixed
5. **Group and order** - Group related bugs that touch the same files or
   modules. Order fix phases so that foundational fixes come first (e.g. fix a
   shared utility before fixing the components that use it). Identify any bugs
   that conflict or where fixing one may resolve another

### Fix Phases

Create one fix phase per bug, or per group of closely related bugs that touch
the same files. Each fix phase should be a self-contained unit of work.

For each fix phase:

- **Phase title** - Name it after what it fixes (e.g. "Fix Phase: Null Reference
  in UserService", "Fix Phase: DeepSource Type Safety Issues in Auth Module")
- **Bugs addressed** - List which bugs from the inventory this phase resolves
- **Scope** - List every file this phase touches and what changes
- **Dependencies** - List which prior phases must be complete before this one
  can start
- **Tasks** - Numbered list of discrete tasks for this phase
- **Related cleanup** - If the fix reveals closely related dead code, naming
  inconsistencies, or minor inefficiencies in the same files, include those. Do
  NOT expand scope beyond the bug's immediate area
- **Deferred tasks** - Any task that CANNOT be completed in this phase because
  it depends on work in a later phase. Log these in a deferred task table:

```markdown
| Deferred Task          | Blocked By   | Resolve In Phase |
| ---------------------- | ------------ | ---------------- |
| Update consumer errors | Consumer fix | Integration      |
```

Keep each fix phase minimal and focused. If a bug is simple, the phase may have
just 2-3 tasks. Do not pad phases with unnecessary work.

### Penultimate Phase: Integration and Wiring

This phase exists to tie everything together. Go through the plan and:

1. **Resolve all deferred tasks** - Every item logged in the deferred task
   tables from the fix phases must be addressed here. No deferred task should
   remain unresolved
2. **Cross-fix verification** - If multiple fixes touched related code, verify
   they work together without conflicts
3. **Consolidation** - If bugs existed because of duplicated logic or
   inconsistent patterns, note the consolidation needed to prevent recurrence

### Final Phase: Testing and Verification

1. **Unit tests** - List every test file to be created or updated. Cover each
   specific bug scenario, edge cases, and boundary conditions
2. **Regression tests** - Based on the blast radius from the diagnosis phase,
   list tests that verify nothing else broke. Regression tests are mandatory for
   every fix
3. **Integration tests** - If fixes spanned multiple modules, tests that verify
   cross-module behaviour
4. **Reproduction verification** - For each bug, document how to reproduce it
   before the fix and verify it is resolved after
5. **Cleanup** - Remove any temporary scaffolding or compatibility code
   introduced during fix phases
6. **Final verification** - The plan must target a fully working,
   production-ready result. Every bug in the inventory must be resolved

## Output Format

Produce a markdown document with the following structure:

```markdown
# Bug Fix Plan: [Brief Description or "Batch Fix"]

## Bug Inventory
| # | Bug Description | Root Cause | Files Affected | Blast Radius |
|---|---|---|---|---|

## First Phase: Diagnosis and Triage
### Dependency Traces
### Root Cause Analysis
### Grouping and Order

## Fix Phase: [Bug/Group Description]
### Bugs Addressed
### Scope
### Dependencies
### Tasks
### Related Cleanup
### Deferred Tasks
| Deferred Task | Blocked By | Resolve In Phase |
|---|---|---|

[Repeat for each fix phase]

## Penultimate Phase: Integration and Wiring
### Deferred Task Resolution
### Cross-Fix Verification
### Consolidation

## Final Phase: Testing and Verification
### Unit Tests
### Regression Tests
### Integration Tests
### Reproduction Verification
### Cleanup

## Full Checklist
[Numbered list of every discrete task across all phases, in order]
```

## Rules

- Do not write implementation code in the plan unless the user's prompt
  explicitly asks you to generate or save files
- All reference material (attached HTML, images, Figma links, external files,
  other project folders) is READ-ONLY. Never modify, move, or delete reference
  material
- Only create or modify files within the active workspace (the directory you are
  currently working in)
- Keep each fix minimal. Do not refactor, add features, or clean up unrelated
  code
- Always run `assistgraph dependents <path> --depth 0` for each proposed fix
  location and account for truncation before describing its blast radius
- Every changed file must have corresponding test coverage in the plan
- Regression tests are mandatory, not optional
- Every deferred task must name the phase where it will be resolved
- The penultimate phase must resolve ALL deferred tasks with none remaining
- If information is missing, list it as an open question rather than guessing
