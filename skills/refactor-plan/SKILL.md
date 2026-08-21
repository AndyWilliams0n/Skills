---
name: refactor-plan
description:
  Use when the user asks to plan a refactor, restructure, cleanup, dependency
  consolidation, module split, file move, or maintainability improvement before
  coding. Produces a structured refactoring plan using assistgraph to identify
  dependencies, circular imports, orphan files, hubs, and module boundaries.
---

# Refactor Plan

You are a planning agent. Your job is to produce a structured, thorough
refactoring plan. Produce a plan only. Do not write implementation code unless
the user's prompt explicitly asks you to generate or save files.

## Delegate evidence collection

The primary model remains the planning agent. Delegate bounded, read-only graph
and source investigation when the task has independent workstreams or enough
evidence to crowd the primary context. Work directly when the scope is small,
tightly sequential, or a suitable worker is unavailable.

Use Luna workers with a GPT Sol or Astra primary. Use Haiku workers with a
Claude Opus or Fable primary. Keep the user-selected primary model and reasoning
mode unchanged.

Give each worker a concrete scope. It may run read-only graph status and bounded
queries, inspect selected source and tests, and return a compact evidence
report. It must follow applicable project rules and use the `assist-graph` skill
when available, otherwise the fallback below. Workers must not build or audit
the graph, modify project rule files, choose final refactor boundaries, or write
the plan. The primary confirms critical evidence and owns architectural
tradeoffs, phase ordering, and the final plan.

## Follow project rules

Before gathering evidence or delegating work, search every project or workspace
in scope for Markdown files whose filename contains `AGENTS` or `RULES`, using a
case-insensitive match. This includes names such as `AGENTS.md`, `RULES.md`, and
`RULES_FOR_AGENTS.md`. Exclude dependency, version-control, build, and generated
directories unless one is part of the requested scope.

Read every matching file that applies to the paths being investigated and
follow its instructions. Treat a rule file as applying to its directory and
descendants unless it defines a different scope. When project rule files
conflict, prefer the closest applicable file unless a higher-priority
instruction requires otherwise. Report a material unresolved conflict instead
of inventing a rule.

Do not create, modify, rename, or delete project rule files unless the user
explicitly asks. Do not invent new project rules or propose a new `AGENTS.md`,
`RULES.md`, or other matching Markdown file as part of an ordinary plan.

## Gather structural evidence with assistgraph

If the `assist-graph` skill is available, apply it and read its `SKILL.md`
before running an AssistGraph command. Treat it as the canonical source for CLI
usage, graph freshness, workspace-write safety, bounded output, and structural
verification.

If the skill is unavailable, use this fallback:

- Run the installed `assistgraph` binary from the workspace root, or use
  `npx -y assistgraph` when the binary is unavailable.
- Run `assistgraph status` first. When the graph is missing or structurally
  stale, run `assistgraph build --no-vault` only when workspace writes are
  permitted, then check status again.
- Use bounded CLI queries. Never read or print `.assist/graph/graph.json`, and
  do not install or configure the optional MCP adapter.
- Check `truncated` on every bounded result. Narrow the query or increase its
  limit before describing the result as complete.
- Use the graph for files, declarations, imports, dependencies, and dependents.
  Inspect source and tests for behavior, responsibilities, and safe boundaries.

Collect relevant cycle, orphan, and community findings. Run the write-producing
audit only when generated analysis output is permitted. For each proposed
boundary, locate its files and declarations, inspect representative file
metadata, map dependencies and dependents, and use path queries where a
connection needs proof. Compare community membership with dependency closure
before proposing moves, splits, or extractions.

## Planning Phases

Work through these phases in order. Each phase should be a clearly labelled
section in your output. The number of implementation phases will vary depending
on the complexity of the refactor.

### First Phase: Setup and Scaffolding

1. **Analyse the dependency graph** - Combine audit findings with focused
   community, file, symbol, dependency, and dependent queries. Identify circular
   dependencies, tightly coupled modules, orphan files, and oversized modules.
   Confirm duplicated behavior from source; the graph cannot detect it
2. **Dependency cleanup** - List packages to remove, upgrade, or consolidate.
   Identify duplicate or overlapping dependencies. Include install/uninstall
   commands
3. **Files and folder structure** - Map out structural changes: files to move,
   rename, split, or merge. Show before and after paths. Create any new folders
   needed
4. **Shared vs feature components** - Identify components that are currently
   feature-scoped but should be promoted to shared, and any shared components
   that are only used once and should be demoted

### Next Phase: Layouts and Components

1. **Target component changes** - For each component being refactored, define
   what changes and why. Focus on the interface (props, inputs/outputs) not
   internal implementation
2. **API design refactoring** - If the refactor touches API layers, define
   endpoint changes, deprecations, and migration paths
3. **Backend to frontend integration changes** - If refactoring crosses the
   stack, map how data flow changes between backend and frontend
4. **Attached resources** - If the user has provided HTML mockups, images, or
   Figma links, use these as the definitive guide for any layout or styling
   changes

### Implementation Phases

Break the remaining work into multiple discrete phases. Each phase should be a
self-contained unit of work that can be completed and verified independently.
Split by module, feature area, or logical boundary - whichever produces the
clearest separation.

For each implementation phase:

- **Phase title** - Name it after what it delivers (e.g. "Implementation Phase:
  Auth Service Consolidation", "Implementation Phase: Dashboard Component
  Split")
- **Scope** - List every file this phase touches, with before/after paths for
  moves and renames
- **Dependencies** - List which prior phases must be complete before this one
  can start
- **Tasks** - Numbered list of discrete tasks for this phase
- **Clean code gains** - Specific improvements in this phase: deduplication,
  naming, dead code removal, simplification
- **Deferred tasks** - Any task that CANNOT be completed in this phase because
  it depends on work in a later phase. Log these in a deferred task table:

```markdown
| Deferred Task            | Blocked By          | Resolve In Phase |
| ------------------------ | ------------------- | ---------------- |
| Update dashboard imports | Dashboard refactor | Integration      |
```

Keep implementation phases granular. Each phase should be completable without
leaving broken code. Order phases so that foundational changes (shared
utilities, core services) come before dependent changes (feature modules that
import them).

### Penultimate Phase: Integration and Wiring

This phase exists to tie everything together. Go through the plan and:

1. **Resolve all deferred tasks** - Every item logged in the deferred task
   tables from the implementation phases must be addressed here. No deferred
   task should remain unresolved
2. **Cross-module wiring** - Update all import paths, re-exports, and barrel
   files affected by moves and renames across phases
3. **Consolidation verification** - Verify that duplicated code identified in
   earlier phases has been properly consolidated
4. **Error handling consistency** - Ensure error handling patterns are
   consistent across all refactored modules

### Final Phase: Polish and Testing

1. **Clean code audit** - Final review of all changes for naming consistency,
   dead code, unnecessary complexity
2. **Consolidation and efficiency** - Verify all merge opportunities have been
   captured: similar utilities, reducible bundle size, simplified dependency
   tree
3. **Maintenance gains** - Document how the refactor improves long-term
   maintainability: clearer module boundaries, better separation of concerns,
   easier testing
4. **Unit tests** - List every test file to be created or updated. Tests are
   especially important during refactoring to prevent regressions. Cover all
   changed components, services, and utilities
5. **Integration tests** - Tests that verify cross-module wiring works correctly
   after structural changes
6. **Documentation** - Document the refactored structure: what changed, where
   things moved, and why
7. **Cleanup** - Remove any temporary scaffolding, unused imports, or
   compatibility shims introduced during implementation phases
8. **Final verification** - The plan must target a fully working,
   production-ready result. No broken imports, no missing references, no partial
   refactors

## Output Format

Produce a markdown document with the following structure:

```markdown
# Refactor Plan: [Scope Description]

## Summary
[2-3 sentence overview of what is being refactored and why]

## Audit Findings
[Key issues from assistgraph audit]

## First Phase: Setup and Scaffolding
### Problem Areas
### Dependency Cleanup
### File and Folder Changes (Before/After)
### Shared vs Feature Components

## Next Phase: Layouts and Components
### Component Changes
### API Refactoring
### Integration Changes
### Resource References

## Implementation Phase: [Module/Area Name]
### Scope (Before/After)
### Dependencies
### Tasks
### Clean Code Gains
### Deferred Tasks
| Deferred Task | Blocked By | Resolve In Phase |
|---|---|---|

[Repeat for each implementation phase]

## Penultimate Phase: Integration and Wiring
### Deferred Task Resolution
### Cross-Module Wiring
### Consolidation Verification
### Error Handling Consistency

## Final Phase: Polish and Testing
### Clean Code Audit
### Consolidation and Efficiency
### Maintenance Gains
### Unit Tests
### Integration Tests
### Documentation
### Cleanup

## Risk Assessment
[What could break, and how to mitigate]

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
- Support every proposed boundary change with the relevant `file`, `deps`, and
  `dependents` results rather than citing the raw graph generically
- Show before/after for every structural change
- Order the checklist so that no task breaks the build when completed in
  sequence
- Every deferred task must name the phase where it will be resolved
- The penultimate phase must resolve ALL deferred tasks with none remaining
- Flag any circular dependency risks introduced by the refactor
- If information is missing, list it as an open question rather than guessing
