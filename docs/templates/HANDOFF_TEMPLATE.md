# HANDOFF_<NNN> — <TITLE>

**Task:** `TASK_<NNN>`  
**Status:** CODEX_HANDOFF  
**Branch:** `<branch>`  
**Base HEAD:** `<sha>`  
**Final pushed HEAD:** `<sha>`

## 1. What was implemented

- <item>
- <item>

## 2. Files changed

```text
<path>
<path>
```

## 3. Exact delivered behavior

<Describe actual behavior, not intention.>

## 4. Tests and checks run

| Command | Result | Notes |
|---|---|---|
| `<command>` | PASS/FAIL | <notes> |

Do not omit failing commands. If a required command was not run, state `NOT RUN` and why.

## 5. Live probes

**Status:** `NONE | PASS | PARTIAL | BLOCKED | FAIL`

If run, record:

- source;
- UTC start/end;
- purpose;
- request/message/record counts where practical;
- authentication used: yes/no and type only, never secrets;
- evidence/manifest path;
- observed limitations.

## 6. Fixtures/evidence created

- <path + what it proves>

## 7. Security/secret review

- runtime directories ignored: PASS/FAIL;
- secret scan/diff review: PASS/FAIL;
- cookies/tokens/private keys committed: MUST BE NO;
- raw captures sanitized before fixture commit: PASS/FAIL/NOT APPLICABLE.

## 8. Deviations from task

`NONE` or exact deviations with reasons.

Do not normalize an architecture change as a harmless deviation.

## 9. Unresolved issues / risks

- <item or NONE>

Use `DECISION_REQUIRED` when architect input is needed.

## 10. Implementation verification level

Mark each accurately:

```text
IMPLEMENTED: YES/NO
UNIT_TESTED: YES/NO
INTEGRATION_TESTED: YES/NO
REPLAY_TESTED: YES/NO/NA
LIVE_VERIFIED: YES/NO/NA
SOAK_TESTED: YES/NO/NA
```

## 11. Git evidence

```text
branch: <branch>
commit(s): <sha list>
remote pushed: YES/NO
working tree at handoff: CLEAN / <explain>
```

## 12. Suggested next action

<Advisory only. Architect decides.>

## 13. Stop acknowledgement

Codex stopped at the assigned task boundary and did not begin the next roadmap phase: `YES/NO`.
