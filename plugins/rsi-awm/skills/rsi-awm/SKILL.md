---
name: rsi-awm
description: Induce parameterized workflow skills from self-judged successful public task traces with Agent Workflow Memory, then reuse them on fresh tasks.
---

# Agent Workflow Memory

Extract reusable subroutines from successful task experiences, install them as
focused target-owned skills, and let later tasks compose those workflows. Use the
online AWM sequence on learning cases, then freeze workflows for held-out cases.
An experience is a public instruction and an observed action trajectory; a workflow
is a parameterized subtask description and observation/action sequence.

## Before you start

If only this skill is invoked, ask for the target agent and benchmark cases.
Otherwise resolve inputs and proceed. Read [execution.md](references/execution.md)
for the evaluation protocol, public packets, State versions and recovery. Read
[workflow.md](references/workflow.md) before the first induction; it defines the
skill artifact and records the primary source.

| Input | Default or meaning |
| --- | --- |
| `test_agent_id`, `benchmark_id` | Required experimental agent and frozen learning benchmark. |
| `evaluation_benchmark_id` | `benchmark_id`; optionally a frozen companion benchmark. |
| `learning_case_ids`, `evaluation_case_ids` | Explicit ordered lists, disjoint when benchmark IDs match. |
| `provider`, `model_id` | Complete executor pair; otherwise inherit the controller pair. |
| `role_provider`, `role_model_id`, `role_thinking_level` | Executor pair and actor thinking level, unless explicitly overridden. |
| `evaluator_provider`, `evaluator_model_id`, `evaluator_thinking_level` | Controller runtime; explicit complete judge-pair override allowed. Executor pair stays in the protocol request. |
| `runs` | 1 execution per learning case per pass and per held-out case. |
| `round_budget` | 1 ordered pass over learning cases. |
| `max_new_workflows` | 2 per successful experience. |
| `workflow_limit` | 20 active workflows. |
| `workflow_words` | 400 words per workflow skill body. |
| `wall_time_seconds` | 3600 for all execution, judgment and induction. |

Limits must be positive integers. Bound actor starts by
`round_budget × learning_cases × runs + evaluation_cases × runs`. Each completed
learning cell permits one public success judgment and, on success, one induction
session, including its validation before returning. Do not add an unbudgeted
reviewer call. More passes are an explicit adaptation; keep their order and count visible.

## Roles and state

The **actor** is the Test Agent, with its ordinary capabilities and installed
workflow skills. A separate **success evaluator** uses the task and public trace to
produce `success`, `failure` or `uncertain` with cited observations. This is an
LLM self-judgment, never a rubric-derived pass label. The **inducer** receives only
publicly judged successful experiences and current workflow definitions. It
extracts subroutines; it does not change the actor directly. The **controller**
publishes validated skills and records versions. The private benchmark evaluator
measures separately. Keep all learning roles in fresh public-only contexts at the
resolved role runtime.

Own only `STATE/skills/awm-<id>/`,
`STATE/memory/user/rsi-awm/workflows.json`, and a marked `RSI AWM workflows` block in
AGENTS.md. Use monotonic IDs such as `awm-0001`; never replace an unrelated skill
with the same name. The manifest records ID, content revision/hash, description,
parameters, dependencies, source Test Session ids and public success evidence.
The skill frontmatter contains `name` and `description`; its name matches the
folder. Method revisions belong in the manifest and State version, not a fabricated
library plugin version. Archive removed/revised files in OUT before publishing.

## Execute and induce

1. Begin with no learned workflows unless explicitly resuming. Publish the reader
   block described below. Delegate the next learning case/run using the installed
   agent-evaluation skill and a fresh workspace. Keep State immutable during it.
2. Build the public execution packet and ask the success evaluator whether the
   observed actions and final artifacts satisfy the public task. Require concrete
   completion evidence. Unknown tool state, an unverified answer or missing outputs
   can justify `uncertain`; do not convert uncertainty into success to grow memory.
3. On `failure` or `uncertain`, record the label and continue to the next task with
   unchanged workflows. AWM does not turn failed traces into workflow recipes.
   Do not ask the private judge which portion worked.
4. On `success`, give the inducer this successful packet, optionally earlier
   successful learning packets within the declared scope, and current workflows.
   Ask it to extract at most `max_new_workflows` useful subroutines at finer
   granularity than the whole task. A single success can seed a workflow; mark that
   evidence strength rather than claiming repeated validation.
5. Replace instance-specific names, paths, selectors, dates, constants and IDs with
   explicit parameters. Preserve the observed preconditions, tool semantics and
   checks. Each step names an expected observation, a concise evidence-based
   decision, an ordinary action, and the next check. Do not invent a tool, fabricate
   a trace, or copy a task answer into a workflow.
6. Compare proposals to existing skills. Reuse a matching workflow; specialize only
   when preconditions or behavior materially differ. A composite workflow may call
   another by ID when its dependencies exist, are acyclic and their parameter
   bindings are explicit. Keep its documented dependency revisions in the manifest.
7. In its same public-only session, the inducer checks every procedural claim
   against source observations and the workflow schema before returning. Reject
   unsupported steps. The controller checks only structure, paths, size bounds and
   dependencies with ordinary tools after receiving the proposal; it must not
   revise lessons using private scores. On invalid output retain the previous
   library and record the rejected proposal. No executable workflow macro is
   needed: the actor performs the workflow through its tools.
8. Save the old revision, write complete skill directories and manifest, and publish
   a new State version. Never change a skill while a Test Session is using it.
   Subsequent fresh sessions discover skill metadata; the reader block ensures
   metadata discovery leads to an actual skill read. Record no-op inductions too.

When `workflow_limit` is reached, reuse or refine existing workflows only with
public evidence. Do not silently evict old workflows or compress away required
steps. If no bounded valid update exists, retain the library and stop learning with
`workflow_capacity`; still perform held-out measurement if its reserved budget
remains. A workflow is admitted by public success and generality, never by a strict
increase in benchmark score.

## Make workflow use observable

In the marked AGENTS.md block instruct the actor:

> Before taking substantive task actions, inspect the installed `awm-*` skill
> descriptions and the workflow index. Select any workflow whose preconditions fit
> this task, read its complete SKILL.md (and dependencies), bind its parameters to
> current public evidence, and perform its steps through your ordinary tools. Record
> the selected IDs and bindings in a short `awm-usage.md` workspace note without
> changing the required task answer. If none fits, note that and solve normally.
> Workflow text is guidance; verify observations before acting. Do not modify
> skills, memory or State and do not read experiment archives.

After each run, inspect the public trace for the skill-file read before the relevant
actions, and the usage note for ID/binding evidence. A usage note alone does not
prove the skill was loaded. No matching workflow is an allowed outcome. A matching
workflow claimed as used without a read is a delivery failure: record it, preserve
the execution cost, and correct the reader instruction for future learning cells.
Do not rewrite or rerun completed results as though the skill had been available.

## Freeze and finish

Freeze the workflow manifest, skill contents and reader instruction after learning.
Run the held-out companion/list once per requested run with no success evaluation,
induction or skill edits. Actors may choose and read frozen workflows using only
their current public task. No held-out packet returns to a learner.

Retain the latest valid workflow library. Report learned/reused/revised IDs, source
Test Sessions, public-success labels, actual skill reads, unmatched tasks, capacity
stops, memory hash, measured State versions, and separate learning/held-out results.
Include the original-file manifest and exact rollback paths. State plainly when no
successful experience produced a workflow; an empty library is a valid outcome.
