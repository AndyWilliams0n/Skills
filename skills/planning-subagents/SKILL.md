---
name: planning-subagents
description: Delegate bounded assistgraph and source investigation to smaller subagents during bug-fix, migration, new-feature, and refactor planning while the stronger primary model owns judgment and the final plan. Use with a Sol or Astra primary and Luna workers, or an Opus or Fable primary and Haiku workers.
---

# Planning subagents

Use this companion skill with `bugfix-plan`, `migration-plan`,
`new-feature-plan`, and `refactor-plan` when smaller subagents are available.
The primary model remains the planning agent.

## Choose whether to delegate

Delegate when the investigation has multiple independent workstreams or enough
graph and source evidence to crowd the primary model's context. Examples include
separate bugs, source and target workspaces, several feature areas, or multiple
refactor boundaries.

Work directly when the scope is small, the investigation is tightly sequential,
or the handoff would cost more context than it saves. Do not block planning when
a requested worker model or subagent interface is unavailable.

Use these worker mappings when the runtime exposes them:

- With a GPT Sol or Astra primary, use Luna workers.
- With a Claude Opus or Fable primary, use Haiku workers.

Keep the primary model and reasoning mode chosen by the user. Do not replace the
primary with a worker model.

## Keep judgment with the primary

The primary must:

- Discover and read the applicable project rule files before delegating work.
- Interpret the request, project rules, constraints, and success criteria.
- Decide whether generated graph output or a graph rebuild is permitted.
- Split the investigation into concrete, non-overlapping assignments.
- Confirm the critical source evidence behind each conclusion.
- Determine root causes, feature boundaries, parity decisions, tradeoffs, risks,
  phase order, and verification requirements.
- Write and check the final plan.

Subagents must not write the final plan or make unreviewed architectural
decisions.

## Delegate bounded evidence collection

Give each worker a narrow, read-only assignment. A worker may:

- Apply the `assist-graph` skill before running structural queries when it is
  available. Otherwise follow the active planning skill's AssistGraph fallback.
- Run read-only status and bounded structural queries requested by the primary.
- Inspect the selected source files, tests, configuration, and manifests.
- Report structural evidence, runtime-relevant source findings, truncation, and
  unanswered questions.

Include the applicable project rules in every assignment. Workers must follow
those rules and report any additional matching rule file they encounter. They
must not create, modify, rename, or delete a project rule file unless the user
explicitly requested that exact change.

Workers must not run `assistgraph build` or `assistgraph audit`, because those
commands write workspace files. If the graph is missing, stale, or legacy, the
worker reports that fact and stops relying on it. The primary decides the next
step under the active planning skill's rules.

Avoid overlapping assignments and repeated repository scans. For a migration,
one worker can map the reference feature while another maps target conventions.
For a bug batch, split workers by unrelated bug or module. For a refactor, split
by independent boundary rather than by arbitrary file count.

## Require compact reports

Ask each worker to return only:

1. Scope investigated.
2. Commands run and graph freshness.
3. Relevant files, declarations, dependencies, and dependents.
4. Source and test evidence with paths.
5. Truncated results, conflicts, risks, and open questions.

Do not ask workers to paste complete command output, reproduce the whole graph,
or draft the final plan. The primary should re-open the few critical files and
resolve disagreements before synthesis.
