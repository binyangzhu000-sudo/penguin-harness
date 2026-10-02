# Workflow artifact

Each `STATE/skills/awm-<id>/SKILL.md` contains a concise, independently usable
subroutine. This is an illustrative shape, not a script:

```markdown
---
name: awm-0001
description: Reconcile keyed records from two public files when stable record IDs and a conflict rule are available.
---

# Reconcile keyed records

Inputs: left_path, right_path, id_field, conflict_rule, output_path.
Preconditions: both inputs can be parsed; id_field exists; the caller supplies the
conflict rule. Ask for the rule when it is missing rather than inventing one.

1. Observe each input's fields and record IDs. Check uniqueness before selecting a
   keyed merge; preserve duplicate records for explicit resolution.
2. Bind the public conflict rule and merge corresponding records with ordinary
   file tools. Do not assume that file order encodes recency.
3. Write output_path and verify every output ID against the inputs and rule.

Completion: the requested output exists and the reconciliation checks are recorded.
Failure boundary: missing IDs or an undefined conflict rule require clarification.
```

Only induce a workflow like this when its actual steps appear in a successful
public trajectory. The example grants no evidence for installing it. Store provenance
in the workflow manifest so the actor does not need to browse old task archives.
For an observation/action workflow include, per step:

| Field | Required content |
| --- | --- |
| Observation | What the actor can verify in the current environment. |
| Decision | Why the next action follows from that observation, without invented hidden reasoning. |
| Action | An available tool operation with parameter bindings. |
| Check | Evidence needed before proceeding or declaring completion. |

Keep parameters explicit; a path or selector from an old task is not a default for
a new task. A workflow may compose an existing workflow by ID with explicit inputs
and completion checks, but circular dependencies invalidate the proposal.

# Primary source and adaptations

Source: Zora Zhiruo Wang, Jiayuan Mao, Daniel Fried and Graham Neubig,
**Agent Workflow Memory**, arXiv:2409.07429 (2024), subsequently ICML 2025.

- Paper: <https://arxiv.org/abs/2409.07429>
- Relevant passages: §2.1 experiences as instructions and observation/action traces;
  §2.2 workflow description and trajectory; §2.3 induction and integration,
  especially online neural success judgment; Appendix A for induction examples.

The online method judges a generated trajectory's success before induction, extracts
finer-grained subroutines and abstracts instance values into parameters. Offline AWM
instead induces from canonical training experiences before test inference. This
skill runs online learning on the supplied learning list and freezes the result for
the companion evaluation. It does not treat hidden benchmark scores as its online
success signal.

Target-owned skills, a reader instruction and trace-verified loading implement the
paper's integration into agent memory without executable workflow macros. This
packaging and the workflow/word caps are harness adaptations. The paper's setting
is web navigation; applying observation/action induction to other tasks should be
reported as a domain adaptation, not an exact benchmark reproduction.
