# 11 — Architect ↔ Codex Work Protocol

## 1. Purpose

The repository is the durable coordination channel between the architect and local Codex.

Chat conversation may initiate work, but the authoritative task specification, implementation handoff and architect review should be persisted in GitHub so context can be recovered without relying on chat memory.

## 2. Responsibility split

### Architect

The architect:

- decides what to build next;
- writes task specifications;
- defines acceptance tests/gates;
- decides architecture and dependencies;
- reviews commits/diffs/test evidence;
- accepts or rejects work;
- updates architecture/decision/state authority;
- issues correction tasks.

### Codex

Codex:

- reads authority before work;
- verifies Git state;
- implements only assigned scope;
- runs required tests/probes;
- creates sanitized fixtures/evidence;
- commits and pushes;
- writes a precise handoff;
- does not self-approve architectural phase completion.

## 3. Repository coordination structure

Use:

```text
docs/tasks/
  TASK_001_....md
  TASK_002_....md

docs/handoffs/
  HANDOFF_001_....md
  HANDOFF_002_....md

docs/reviews/
  REVIEW_001_....md
  REVIEW_002_....md

docs/templates/
  TASK_TEMPLATE.md
  HANDOFF_TEMPLATE.md
  REVIEW_TEMPLATE.md
```

Task numbers are monotonic project-wide.

## 4. Task lifecycle

```text
ARCHITECT_DRAFT
    -> READY_FOR_CODEX
    -> CODEX_IN_PROGRESS
    -> CODEX_HANDOFF
    -> ARCHITECT_REVIEW
       -> ACCEPTED
       -> REWORK_REQUIRED
       -> BLOCKED_DECISION_REQUIRED
```

A task can be superseded only by an architect record that identifies the replacement.

## 5. Task document ownership

Architect task specification is treated as immutable implementation authority for that task after `READY_FOR_CODEX`.

Codex does not rewrite acceptance criteria to match its implementation.

If clarification/design change is necessary, Codex records it in the handoff/blocker report and stops the conflicting part. Architect updates/supersedes the task.

## 6. Required task contents

Every task must state:

- task ID/title;
- phase;
- repository/branch policy;
- verified base/expected HEAD when relevant;
- purpose;
- in-scope work;
- explicitly out-of-scope work;
- authority documents to read;
- files/modules expected to change;
- required implementation behavior;
- required tests;
- live-provider calls allowed/forbidden;
- evidence/artifacts required;
- security constraints;
- acceptance criteria;
- commit/push/readback requirements;
- next-step prohibition if appropriate.

## 7. Codex preflight report

Before changing files Codex should capture in its own working notes/handoff:

```text
branch
local HEAD
remote target HEAD
git status
required authority read
unexpected existing changes: yes/no
```

If base assumptions are wrong, do not paper over them.

## 8. Working branch

Default:

```text
codex/<phase>-<task-slug>
```

Codex pushes the branch. The architect may review branch commits directly or request a PR depending on task size.

Direct pushes to `main` by Codex are not the default workflow.

## 9. Commit discipline

Commits should be reviewable and scoped.

Do:

- meaningful message;
- implementation + its tests together where practical;
- no unrelated formatting avalanche;
- no generated/runtime data unless required and sanitized;
- no secret-bearing artifacts.

Do not:

- force push;
- rewrite accepted history;
- mix several roadmap phases in one commit;
- hide failing tests behind unrelated changes.

## 10. Handoff document

Codex creates a handoff after implementation/testing and before declaring the task ready for architect review.

Required sections:

1. task/base/branch;
2. implementation summary;
3. files changed;
4. exact behavior delivered;
5. tests run — exact commands and results;
6. live probes — exact scope/results, or `NONE`;
7. evidence/fixture paths;
8. security/secret check;
9. deviations from task — ideally none;
10. unresolved risks/blocks;
11. commit SHA(s);
12. remote push/readback state;
13. recommended next action (advisory only).

Never say “fully working” when live behavior was not tested.

## 11. Architect review

Architect review checks:

- task scope compliance;
- architecture compliance;
- semantic correctness;
- test adequacy;
- error/reconnect behavior;
- security/access boundaries;
- diff quality;
- live evidence when required;
- no hidden expansion.

Review result is persisted as:

```text
ACCEPTED
REWORK_REQUIRED
BLOCKED_DECISION_REQUIRED
```

A review may list blocking and non-blocking findings.

## 12. Rework

For `REWORK_REQUIRED`:

- architect creates review findings or correction task;
- Codex fixes on the same branch unless instructed otherwise;
- new/updated regression tests are mandatory for defects;
- Codex pushes a new commit and updates/new handoff;
- architect re-reviews.

Do not resolve findings by weakening acceptance criteria.

## 13. Decision requests

Codex must escalate rather than invent when it discovers:

- source protocol materially different from architecture;
- unclear terms/access boundary;
- source requires authentication unexpectedly;
- required rule semantics unavailable;
- proposed dependency changes architecture/security significantly;
- database/data-contract change is needed;
- task acceptance criteria conflict;
- live evidence disproves an assumption.

Report:

```text
DECISION_REQUIRED
Observed evidence:
Impact:
Options:
Codex recommendation (optional):
Work safely completed so far:
Blocked work:
```

Architect makes the decision and records it in `DECISION_LOG.md` when material.

## 14. Provider/live calls

Default is **no unbounded external probing**.

A task explicitly states whether live calls are:

```text
FORBIDDEN
BOUNDED_PROBE_ALLOWED
NORMAL_READ_ONLY_ALLOWED
```

For bounded probes, the task specifies source/purpose/duration or record limits as practical.

No task under current roadmap authorizes wagering.

## 15. Project state

`docs/PROJECT_STATE.md` is the compact recovery cursor.

It records:

- current phase;
- current task;
- accepted through;
- blocked items;
- authoritative latest decisions;
- next architect action;
- next Codex action if any.

Codex changes it only when the task explicitly instructs it. Architect owns final state transitions.

## 16. GitHub issues/PRs

Issues/PRs may supplement task docs, but they do not silently override authority docs.

Use PRs when:

- diff is large;
- inline review is valuable;
- CI status needs a review surface;
- architect requests one.

Small bootstrap tasks may be reviewed by branch/commit directly.

## 17. Remote readback

For important tasks, acceptance requires remote evidence rather than trusting local state.

Codex should provide pushed SHA. Architect may independently fetch:

- commit metadata;
- changed files/diff;
- handoff;
- tests/CI;
- authority consistency.

If local SHA was not actually pushed, task is not ready for review.

## 18. No next-step drift

When a task says “do not proceed”, Codex stops after the handoff. It does not opportunistically implement the next roadmap phase.

## 19. Context recovery protocol

When a new Codex/ChatGPT session takes over:

1. read `README.md` + `AGENTS.md`;
2. read all authority docs named by current task;
3. read `PROJECT_STATE.md`;
4. read latest relevant `DECISION_LOG.md` entries;
5. read current task;
6. read previous task handoff/review if it is a dependency;
7. verify remote HEAD;
8. only then continue work.

## 20. Architect acceptance language

Only use `ACCEPTED` when required evidence passes.

Use precise partial states such as:

```text
IMPLEMENTED_NOT_LIVE_VERIFIED
LIVE_PROBE_PARTIAL
BLOCKED_BY_SOURCE
TESTS_PASS_LIVE_NOT_RUN
REWORK_REQUIRED
```

This project values accurate state over optimistic status wording.
