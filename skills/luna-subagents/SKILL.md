---
name: luna-subagents
description: "Use GPT-5.6 Luna Max Fast subagents as code readers, writers, and executors while a GPT-5.6 Sol primary agent at high, xhigh, max, or stronger plans, orchestrates, reviews, and teaches. Must always apply to code-changing work when those conditions are met."
---

# Luna subagents

Apply this skill whenever both conditions are true:

- The task will create, modify, or delete source code, tests, scripts, dependency manifests, build configuration, or other executable project files.
- The primary agent is GPT-5.6 Sol at `high`, `xhigh`, `max`, or a stronger quality setting exposed by the runtime.

Do not apply it to read-only investigation, planning, ordinary questions, or documentation-only changes.

## Role split

The primary Sol agent is the lead. It must:

- Interpret the request, constraints, and success criteria.
- Form the plan and divide the work into bounded assignments.
- Direct Luna subagents, review their reports and diffs, and decide follow-up work.
- Explain decisions, results, tradeoffs, and lessons to the user.

Sol may inspect Luna's diffs and evidence to review the work, but it must not act as the primary code reader or executor while Luna is available. Sol must not directly edit project code, run implementation commands, or take over Luna's testing work under this skill.

Luna subagents are the workers. They must:

- Read and trace the relevant repository code.
- Implement the requested change and edit the project files.
- Add or update tests when appropriate.
- Run lint, tests, builds, runtime checks, and other proportionate verification.
- Report the files changed, checks run, results, failures, and unresolved risks to Sol.

## Required execution

Before completing the code change, use at least one subagent configured with:

- Model: `gpt-5.6-luna`
- Reasoning effort: `max`
- Execution mode: Fast mode for the Luna subagent only. If the runtime labels Fast execution as the priority service tier, use that tier for the Luna subagent.

Keep the primary Sol agent in the mode and reasoning effort selected by the user. Never enable Fast mode on the primary agent as part of this skill. Do not substitute Sol for Luna merely to obtain Fast mode.

If the subagent interface does not expose a Fast or priority selector because its execution tier is fixed by the runtime, use the runtime's configured priority tier for Luna without changing the primary agent. If fast or priority execution is neither selectable nor guaranteed for Luna, state the limitation before editing and ask whether to continue without it.

Give every Luna subagent a concrete, bounded task that contributes to the result. For a small change, one Luna subagent should inspect, implement, and verify the complete change. For larger work, divide source investigation, disjoint implementation areas, test work, or independent review across multiple Luna subagents.

Keep simultaneous file ownership disjoint. When edits would overlap, give one Luna ownership of the files and use other Luna agents for investigation, testing, or review.

Sol must review the Luna output and request follow-up work when the evidence is incomplete. Sol remains accountable for the integrated result, but Luna performs the execution and verification.

If Luna is unavailable, stalls, or fails, Sol must not silently take over the code change. Retry with a narrower Luna assignment when reasonable. Otherwise, state the limitation and ask whether the user wants to permit an exception.

In the final response, Sol should teach from the completed work. Explain what changed, why the approach was chosen, how it was verified, and how the Luna subagents contributed.
