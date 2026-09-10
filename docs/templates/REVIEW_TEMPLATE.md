# REVIEW_<NNN> — <TITLE>

**Task:** `TASK_<NNN>`  
**Reviewed branch:** `<branch>`  
**Reviewed commit:** `<sha>`  
**Review status:** `ACCEPTED | REWORK_REQUIRED | BLOCKED_DECISION_REQUIRED`  
**Architect:** ChatGPT / project architect

## 1. Scope reviewed

- task specification;
- Codex handoff;
- commit/diff;
- tests/CI evidence;
- live probe evidence where required;
- applicable authority docs and acceptance gates.

## 2. Summary verdict

<Concise verdict and why.>

## 3. Acceptance-gate results

| Gate | Result | Evidence / notes |
|---|---|---|
| Authority compliance | PASS/FAIL/NA | |
| Git/repository hygiene | PASS/FAIL/NA | |
| Secret/data hygiene | PASS/FAIL/NA | |
| Build/tooling | PASS/FAIL/NA | |
| Data contracts | PASS/FAIL/NA | |
| Source connector | PASS/FAIL/NA | |
| Matching safety | PASS/FAIL/NA | |
| Arbitrage mathematics | PASS/FAIL/NA | |
| Lifecycle/alerts | PASS/FAIL/NA | |
| Async/reliability | PASS/FAIL/NA | |
| Storage/replay | PASS/FAIL/NA | |
| Windows operability | PASS/FAIL/NA | |
| Live evidence | PASS/FAIL/NA | |
| Research validity | PASS/FAIL/NA | |
| Documentation/handoff | PASS/FAIL/NA | |

## 4. Blocking findings

### B-001 — <title>

**Severity:** BLOCKER / MAJOR  
**Evidence:** <file/line/test/log/behavior>  
**Required correction:** <exact requirement>  
**Acceptance test:** <how architect will verify>

Or `NONE`.

## 5. Non-blocking findings

### N-001 — <title>

**Severity:** MINOR  
**Recommendation:** <action/defer reason>

Or `NONE`.

## 6. Semantic safety check

Explicitly state whether this change can create/alter:

- event matching;
- market matching;
- settlement compatibility;
- quote orientation;
- fee/depth math;
- stale gating;
- opportunity lifecycle.

For every `YES`, record the evidence/tests reviewed.

## 7. Security/access check

State whether review found:

- secrets/runtime data in diff;
- access-control bypass behavior;
- unauthorized authenticated endpoint use;
- automated wagering/order placement;
- unclear terms boundary.

Expected result for acceptance: none, unless an explicitly approved future architecture says otherwise.

## 8. Live verification status

`NOT_REQUIRED | PASS | PARTIAL | BLOCKED | FAIL`

<What was actually live-verified and what remains simulated.>

## 9. Decision

### If ACCEPTED

<State exact task capability accepted and any explicit deferred minor items.>

### If REWORK_REQUIRED

<State correction scope; Codex must not proceed to next phase.>

### If BLOCKED_DECISION_REQUIRED

<State the architecture/source decision needed before more implementation.>

## 10. Project-state action

- `PROJECT_STATE.md` update required: YES/NO
- roadmap phase transition: <none / exact transition>
- next task to issue: <task or NONE>
