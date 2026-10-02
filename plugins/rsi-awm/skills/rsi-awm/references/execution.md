# AWM execution contract

Read once before running AWM. Use the existing `agent-evaluation` skill for
private scoring; keep its protocol and installed files unchanged. The launch
requirements below belong to this AWM invocation and must be included in each
evaluator prompt; they do not require modifying Agent Tuning. The controller owns loops and
memory; fresh public-only workers perform learning. These are instruction-only
access boundaries, not a sandbox. Hashes detect changes; they do not prevent reads.

## Inputs, runtime and paths

Require `test_agent_id`, `benchmark_id`, ordered `learning_case_ids` and
`evaluation_case_ids`. `evaluation_benchmark_id` defaults to `benchmark_id`.
Resolve each list in its respective benchmark. Lists must be unique and disjoint
only when the benchmark IDs match. An explicit empty evaluation list means
learning only. `runs=1` unless supplied; all counts/budgets are positive integers.

Use the explicit complete `(provider, model_id)` executor pair, or inherit both
from the controller Environment when neither is supplied. Reject a half pair.
Read actor `model.thinking_level` from system_config.yaml, default `medium` only if
absent. Preserve that setting. `role_provider`/`role_model_id` default together to
the executor pair; explicit overrides require both. `role_thinking_level` defaults
to actor thinking. Record controller runtime separately. `evaluator_provider` and
`evaluator_model_id` default together to the controller pair, and
`evaluator_thinking_level` defaults to controller thinking. Explicit evaluator pair
overrides require both fields. Spawn evaluator workers with this evaluator runtime,
never automatically with the executor or learning-role runtime. The YAML request
inside that worker still carries the executor pair. Judge and actor models may
differ, but both choices must be recorded before the first execution.
Record requested and actual identities separately; do not infer judge identity from
the actor identity fields in the protocol. Use transport-provided child Session metadata for actual provider/model, and record the spawn thinking setting separately. If transport metadata is unavailable, report requested runtime with actual identity unverified. Never inspect evaluator messages or private traces just to establish its runtime.

```text
TARGET = <Environment App Data Dir>/agents/<test_agent_id>
STATE = <target>/agent_state
BENCHMARK = <app_data_dir>/benchmarks/<benchmark_id>
EVALUATION_BENCHMARK = <app_data_dir>/benchmarks/<evaluation_benchmark_id>
OUT = <target>/scratchpad/rsi/awm/<experiment_id>
```

Use a new UTC timestamp-and-suffix experiment ID, or explicitly resume an existing
one. Reject traversal, separators in IDs and symlink escapes. Require the existing
agent, public statement files and frozen benchmark. Accept `published`, or a
caller-designated immutable draft revision. Do not create an agent or benchmark or
require a Formal Baseline. Inspect only the listed learning statements and config;
record held-out identity/revision without opening its case content until learning
ends. Read only this target and listed public cases, OUT and bound Test Sessions.
Never scan project/home/disk, other agents or unrelated traces. Never read rubrics,
Gold answers, evaluator state/workspace/trace, private rationale, vaults or secrets.

## Runtime spelling and launches

This repository's `run_subagent` tool uses `prompt`, `agent_id`, `provider`,
`model_id`, `thinking_level`. The SDK/server session API uses `agentId`, `provider`,
`modelId`, `thinkingLevel`. Some exposed tool adapters also use camelCase: inspect
the active tool schema and use its spelling, never send both aliases. The YAML
inside an evaluation prompt always keeps `model_id`.

Launch evaluator and learner sessions on the controller Agent, which has the
required skills, not on the Test Agent. Pass only their scoped prompt and artifact
paths. Penguin subagents start fresh conversations; do not paste parent history
into them. They may share agent instructions/workspace, so explicitly exclude
controller logs and private results from learning roles. If another runtime
inherits conversation history, use a fresh CLI session instead or stop.

CLI flags are verified in `packages/cli/src/commands/run.ts`; tool argument mapping
is in `packages/core/src/environment/tools/run-subagent.ts`. Use an already verified
launcher. Do not install one or start a dev server. For a fresh learning role:

```bash
PROJECT_DIR="<Environment App Data Dir>"
PROJECT_ID="$(basename "$PROJECT_DIR")"
PENGUIN_HOME="$(dirname "$PROJECT_DIR")"
export PENGUIN_HOME
penguin run --project-id "$PROJECT_ID" --agent-id "<controller_agent_id>" \
  --workspace "<absolute_public_packet_workspace>" \
  --provider "<role_provider>" --model-id "<role_model_id>" \
  --thinking "<role_thinking_level>" --approve allow-all \
  --message "<scoped role instruction and exact public artifact paths>"
```

Verify the derived project basename and home dirname before launch. Omit `--session`
to obtain a fresh conversation. Use existing authorization/approval settings. The
agent-evaluation worker launches the actor similarly, with the target agent, its
unique public workspace, executor runtime, actor thinking level and
`--source benchmark`; its message is “Read README.md in the current Workspace and
complete the task exactly as specified there.” Keep benchmark execution delegated.
Explicit `--thinking` avoids inheriting the controller's different thinking level.
For stored actor `none`, omit the unsupported CLI `--thinking none` flag and
clear only `PENGUIN_SESSION_ID` for this launch (`env -u PENGUIN_SESSION_ID penguin
run ...` on POSIX; temporarily remove and restore that variable in PowerShell).
Keep all explicit actor/project/workspace/model flags and the connection and
authentication environment. This prevents caller-level inheritance, allowing core
to read the target's unchanged `none` configuration. Do not change Agent Tuning.
For a learning/evaluator role requesting `none`, use a fresh CLI role session with
this same exception only when its Agent configuration already says `none`;
otherwise stop rather than silently choosing another level or editing the Agent.

## One evaluation cell

The controller needs fresh delegation and installed agent-evaluation. Read that
skill, then send this scoped request; learning pass and cell remain in the controller ledger:

```text
Use the existing agent-evaluation Skill. Execute and privately score exactly this one cell.
For this AWM evaluation, parse the complete target YAML for version and model.thinking_level.
Pass --thinking with the parsed selectable level when launching the actor.
For stored none, omit --thinking and clear only PENGUIN_SESSION_ID for that command,
keeping the explicit project/agent/workspace/model/source flags and authentication.
Keep the launch arguments and pre/post config values; do not alter any installed Skill.
Output only the original evaluation protocol YAML. A runtime mismatch stops the cell;
do not relaunch a completed actor merely to correct a report field.
protocol_version: 1
case_id: <case_id>
run: <one_based_run>
expected_version: <current_state_version>
test_agent_id: <test_agent_id>
benchmark_id: <learning_or_evaluation_benchmark_id_for_this_cell>
provider: <executor_provider>
model_id: <executor_model_id>
```

Validate all streamed/final worker text as one plain YAML document before using
fields. No narration, fences, scoring explanation or extra private feedback. Ask
the same worker to resend an existing result for a formatting repair; never rerun
or rescore. Success requires `protocol_version: 1`, `status: ok`, `case_id`, `run`,
`agent_id`, `expected_version`, actual `provider`, `model_id`, configured
`thinking_level`, finite `score` in 0..100, `cost` (number/null), nonnegative integer
`duration_ms`, and `session_id`. Match identities, State version and runtime. A verified actual-runtime mismatch invalidates the cell and stops the experiment without a replacement actor run. A report-only transcription error may be corrected by the same evaluator only
when its saved launch arguments and configuration prove the actual execution;
otherwise stop with the invalid cell. Never modify the evaluator Skill to recover.

Failure uses `status: failed`, request identities and `failure_code`, without
score/cost/duration/session id. Correct `invalid_request`; stop on `version_changed`
or `benchmark_invalid`. For `evaluation_failed`, the same evaluator may repair a
launch only if evidence proves the actor did not start and a specific correction
exists. Never repeat an unchanged launch, retry an ambiguous start or score an
infrastructure error as zero. If the actor completed before the judge failed (for
example, judge rate limiting), preserve that actor's workspace and Test Session.
Continue only the existing worker's scoring of that bound execution when the
installed protocol supports it; never send the cell to a replacement worker that
would launch another actor. Otherwise record the cell as infrastructure failure,
leave its score absent and stop. Do not discard successful public execution evidence
or call the already-spent actor execution an unstarted attempt. Wrong/missing task output is scored behavior, not a
launch retry. Every actual or ambiguous actor start consumes budget; hard wall-time
and any caller cost/token cap also bound repairs and role calls.

## Give learners public evidence

Resolve the returned **Test Session** id only under TARGET/traces. Bind its root
metadata to the requested actor, runtime and unique workspace under
TARGET/workspaces. Read that root and directly referenced Test child sessions,
never the evaluator's trace. Public workspace artifacts must not escape via links.

Make one public packet per cell: statement, observed actions and tool results,
final answer/artifacts, public execution feedback, source Test Session id and
artifact/trace hashes. Link immutable public artifacts instead of duplicating large
files. Preserve available action evidence; label missing evidence and never invent
hidden reasoning. A score alone is insufficient to learn.

Store private numeric benchmark results separately in OUT/evaluations for reporting.
Do not give scores, rubric-derived labels or private rationale to learners or use
them for workflow admission, stopping or version selection. Fresh learning workers use
public evidence and self-judgment only. The controller never writes a lesson itself
after seeing private scores. If private information leaks to a learner, stop as
contaminated and preserve the last uncontaminated memory.

## Publish and recover

Only the controller edits method-owned memory/skills, a marked AGENTS.md block and
the top-level State version. Preserve other settings and capabilities. Actors read
the workflow index and installed AWM skills only, avoid OUT/other cases, and
leave their State unchanged. Disable
no unrelated capability silently; stop if an automatic learning mechanism prevents
State from remaining fixed during execution.

Before edits, record original bytes and absent paths for owned files in OUT/originals.
Record the complete non-secret State hash/version (version defaults to 1 if absent).
Archive each memory revision and created-path list under OUT/versions. Publish only
between runs; every actor-visible change gets a previously unused increasing State
version, including temporary context changes/removal. Compare State and public
statement hashes before/after each cell. They detect drift, not access isolation.

On invalid updates, restore the preceding owned content and remove only newly
created paths. Record the rollback and content/version mapping, verify hashes, and
never overwrite concurrent edits. Keep allocated version IDs even after rollback.
Resume only after checking inputs, frozen hashes, completed cells and remaining
budget; ambiguous in-flight cells require resolution, not automatic rerun.

Cleanup removes temporary injection files/blocks only. Preserve Test Sessions,
workspaces, traces, public packets, snapshots and OUT; never call session deletion.
Retain the AWM workflow library and reader instruction.
A cleanup State revision that was not executed is unmeasured.

## Freeze and report

Freeze learned memory/settings before reading held-out statements. Run
`evaluation_case_ids × runs` in `evaluation_benchmark_id`, once per cell with fresh
sessions/workspaces. No reflection, curation, memory writes, score selection or
failure retries. Task-specific retrieval from frozen memory is allowed; publish it
serially if it changes State, and record its separate context version. Never carry
held-out artifacts to another case or a learner.

Keep records compact: one experiment.json with inputs/runtime/budgets/splits and
original/final hashes, one events.jsonl for cells/updates/rollbacks and budget use,
one raw protocol YAML and public packet per cell, and only method-required memory
revisions. A two-case smoke test needs no extra planning documents or review loop.
Combine independent file operations; do not spend turns polishing experiment logs.

Finish OUT/report.md with completed/missing cells, Test Session ids, memory and
State revisions, learning and held-out scores separately, known costs and unknown
cells, elapsed time, role calls,
stop reason, rollback paths and limitations. Report a complete-matrix average only
when all requested cells are valid; label partial results. Preserve caller-provided
baseline results without rerunning them. Never put mixed versions, subsets,
self-judgments into the benchmark's Formal Scoreboard; do
not modify benchmark files.
