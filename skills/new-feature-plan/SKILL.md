---
name: new-feature-plan
description:
  Use when the user asks to plan a new feature, implementation plan, feature
  build, or feature rollout before coding. Produces a structured implementation
  plan using assistgraph to understand existing dependencies, conventions, file
  structure, and integration points.
---

# New Feature Plan

You are a planning agent. Your job is to produce a structured, thorough
implementation plan for a new feature. Produce a plan only. Do not write
implementation code unless the user's prompt explicitly asks you to generate or
save files.

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
the graph, modify project rule files, choose final feature boundaries or
architecture, or write the plan. The primary confirms critical evidence and
owns tradeoffs, phase ordering, and the final plan.

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
  Inspect source, tests, routes, configuration, and manifests for behavior.

Map repository communities and analogous features, locate relevant public
declarations and extension points, and inspect representative file metadata.
Trace the dependency direction between the proposed feature, shared services,
and existing consumers. Check relevant cycles before proposing cross-community
imports. Run the write-producing audit only when the request requires a broader
structural review and generated analysis output is permitted.

## Planning Phases

Work through these phases in order. Each phase should be a clearly labelled
section in your output. The number of implementation phases will vary depending
on the complexity of the feature.

### First Phase: Setup and Scaffolding

1. **Analyse the dependency graph** - Use communities, files, symbols, and
   representative dependency closures to identify existing feature modules,
   shared components, services, and utilities
2. **Packages and dependencies** - List any new packages or dependencies
   required. Check if similar packages are already in use and prefer those.
   Include install commands
3. **Files and folder structure** - Map out every new file and folder to be
   created. Follow the existing project conventions exactly as shown in the
   graph
4. **Shared vs feature components** - Clearly separate which components belong
   in `shared/` and which are scoped to this feature. Reference existing shared
   components that should be reused

### Next Phase: Layouts and Components

1. **Base pages and component layouts** - Define the page structure, layout
   components, and component hierarchy
2. **API design** - Define endpoints, request/response shapes, error handling
   patterns. Follow existing API conventions from the graph
3. **Backend to frontend integration** - Map API endpoints to frontend service
   calls, state management, and data flow
4. **Attached resources** - If the user has provided HTML mockups, images, or
   Figma links, use these as the definitive guide for layout, workflow, and
   styling decisions

### Implementation Phases

Break the remaining work into multiple discrete phases. Each phase should be a
self-contained unit of work that can be completed and verified independently.
Split by page, feature area, or logical boundary - whichever produces the
clearest separation.

For each implementation phase:

- **Phase title** - Name it after what it delivers (e.g. "Implementation Phase:
  User Profile Page", "Implementation Phase: Search API Endpoints")
- **Scope** - List every file this phase touches and what changes
- **Dependencies** - List which prior phases must be complete before this one
  can start
- **Tasks** - Numbered list of discrete tasks for this phase
- **Deferred tasks** - Any task that CANNOT be completed in this phase because
  it depends on work in a later phase. Log these in a deferred task table:

```markdown
| Deferred Task       | Blocked By   | Resolve In Phase |
| ------------------- | ------------ | ---------------- |
| Link search results | Profile page | Integration      |
```

Keep implementation phases granular. A complex feature with 5 pages should have
at least 5 implementation phases, not 1. Each phase should be completable
without leaving broken code.

### Penultimate Phase: Integration and Wiring

This phase exists to tie everything together. Go through the plan and:

1. **Resolve all deferred tasks** - Every item logged in the deferred task
   tables from the implementation phases must be addressed here. No deferred
   task should remain unresolved
2. **Cross-feature wiring** - Connect all pages, routes, navigation, and shared
   state that span multiple implementation phases
3. **End-to-end data flow** - Verify the complete data flow from backend to
   frontend works across all integrated components
4. **Error handling and edge cases** - Ensure error states, loading states,
   empty states, and edge cases are handled consistently across all phases

### Final Phase: Polish and Testing

1. **Page workflows** - Document the complete user flow: entry points,
   navigation, form submissions, success/error states, edge cases
2. **Unit tests** - List every test file to be created. Cover components,
   services, utilities, and integration points
3. **Integration tests** - Tests that verify cross-phase wiring works correctly
4. **Documentation** - Document the new feature: what it does, where it lives in
   the workspace, its components, and its dependencies
5. **Cleanup** - Remove any temporary scaffolding, unused imports, or
   placeholder code introduced during implementation phases
6. **Final verification** - The plan must target a fully working,
   production-ready feature. No placeholders, no TODO stubs, no partial
   implementations

## Output Format

Produce a markdown document with the following structure:

```markdown
# Feature Plan: [Feature Name]

## Summary
[2-3 sentence overview]

## First Phase: Setup and Scaffolding
### Dependencies
### File and Folder Structure
### Shared vs Feature Components

## Next Phase: Layouts and Components
### Page Layouts
### API Design
### Backend to Frontend Integration
### Resource References

## Implementation Phase: [Area/Page Name]
### Scope
### Dependencies
### Tasks
### Deferred Tasks
| Deferred Task | Blocked By | Resolve In Phase |
|---|---|---|

[Repeat for each implementation phase]

## Penultimate Phase: Integration and Wiring
### Deferred Task Resolution
### Cross-Feature Wiring
### End-to-End Data Flow
### Error Handling

## Final Phase: Polish and Testing
### Page Workflows
### Unit Tests
### Integration Tests
### Documentation
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
- Name the focused assistgraph queries and representative source files that
  support each claimed existing pattern; do not cite the raw graph generically
- Every file in the plan must have a clear purpose stated
- Every deferred task must name the phase where it will be resolved
- The penultimate phase must resolve ALL deferred tasks with none remaining
- Flag any circular dependency risks
- If information is missing, list it as an open question rather than guessing
