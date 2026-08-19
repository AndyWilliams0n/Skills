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
  structure is usable. Open source before relying on an exact location.
- If `structurallyChanged`, `added`, or `removed` is non-empty, rebuild before
  relying on dependency results.
- Run `assistgraph stats` once. When inspecting the first representative file,
  a current graph should expose `symbols`, `structureHash`, and import
  `rawSpecifier`, `bindings`, and `location`. If those fields are absent,
  rebuild the legacy 1.0 graph with the current CLI.

Use this sequence to learn how the codebase already implements similar work:

1. Find analogous features, folders, and entrypoints:

   ```bash
   assistgraph files <feature-or-domain-term> --limit 100
   assistgraph communities --limit 200
   assistgraph communities <community-id> --limit 500
   ```

2. Locate existing public APIs and declarations the new feature can reuse:

   ```bash
   assistgraph symbols <service-component-hook-or-type> --limit 100
   assistgraph symbols <term> --path <likely-folder> --exported --limit 100
   ```

3. Inspect representative entrypoints and shared modules. `file` returns
   declarations, signatures, direct imports, imported/local aliases, source
   spans, and direct dependents:

   ```bash
   assistgraph file <representative-path> --limit 200
   ```

4. Map the complete internal dependency set of an analogous feature and the
   consumers of any shared extension point:

   ```bash
   assistgraph deps <feature-entrypoint> --depth 0 --limit 500
   assistgraph dependents <shared-extension-point> --depth 0 --limit 500
   assistgraph path <feature-entrypoint> <shared-service> --limit 500
   ```

5. Use `assistgraph cycles --limit 200` before proposing new cross-community
   imports. Run `assistgraph audit` only when the request includes a broader
   architecture or codebase-health assessment and generated analysis output is
   permitted; it writes `.assist/graph/audit.md`.

Every bounded result includes truncation metadata. If `truncated` is true,
increase the limit up to 2000 or narrow the file, path, symbol, or community
query. Never infer conventions or complete scope from a truncated result.

The graph tells you where patterns and boundaries are, not how they behave.
Open the representative source files selected above to verify APIs, runtime
flow, state management, styling, error handling, configuration, and tests. Use
text search for call sites and symbol usages because assistgraph indexes
declarations but does not resolve references or function calls.

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
