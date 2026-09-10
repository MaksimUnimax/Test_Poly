# TASK_<NNN> — <TITLE>

**Status:** READY_FOR_CODEX  
**Phase:** <PHASE>  
**Architect owner:** ChatGPT / project architect  
**Implementation owner:** Codex  
**Repository:** `MaksimUnimax/Test_Poly`  
**Base branch:** `main`  
**Expected base HEAD:** `<SHA or VERIFY_LIVE_HEAD>`  
**Work branch:** `codex/<phase>-<task-slug>`

## 1. Purpose

<State exactly why this task exists and what project risk/capability it addresses.>

## 2. Authority to read in full

Before editing, read:

- `README.md`
- `AGENTS.md`
- `docs/PROJECT_STATE.md`
- `docs/DECISION_LOG.md`
- <task-specific authority docs>

Do not start implementation until the repository preflight below is complete.

## 3. Repository preflight

Codex must:

1. `git status --short`;
2. record current branch/HEAD;
3. `git fetch origin`;
4. verify expected base against current `origin/main`;
5. preserve/report unexpected local changes;
6. create/switch to the required work branch;
7. never force push.

## 4. In scope

- <item>
- <item>

## 5. Explicitly out of scope

- <item>
- no automatic betting/order placement;
- no geoblock/access-control/anti-bot bypass;
- no architecture changes not explicitly authorized by this task.

## 6. Required implementation

### 6.1 <Component>

<Exact behavior and interfaces.>

### 6.2 <Component>

<Exact behavior and interfaces.>

## 7. Data/contracts

<State canonical types, timestamps, Decimal semantics, persistence changes and migration requirements.>

## 8. Error/reconnect behavior

<State expected failure modes and safe behavior.>

## 9. Tests required

Codex must add/run:

- <unit tests>;
- <contract tests>;
- <integration/replay tests>;
- <negative cases>.

Exact project-wide commands should be used once tooling is established.

## 10. Live external calls

**Policy:** `FORBIDDEN | BOUNDED_PROBE_ALLOWED | NORMAL_READ_ONLY_ALLOWED`

If allowed:

- source: <source>;
- purpose: <purpose>;
- bounds: <duration/request/data bound>;
- authentication: <none/explicitly authorized>;
- required manifest/evidence: <paths>.

No wagering is authorized.

## 11. Security requirements

- no secrets/cookies/private profiles in Git;
- redact network headers/bodies as required;
- TLS verification stays enabled;
- runtime data stays in ignored paths;
- run/perform required secret check before push.

## 12. Deliverables

- <code/config/scripts>
- <tests/fixtures>
- `docs/handoffs/HANDOFF_<NNN>_<slug>.md`

## 13. Acceptance criteria

The task passes only if:

1. <criterion>;
2. <criterion>;
3. required tests pass;
4. relevant gates in `docs/12_ACCEPTANCE_GATES.md` pass;
5. pushed commit SHA is available for architect review;
6. handoff accurately distinguishes implemented/tested/live-verified/deferred.

## 14. Commit / push / handoff

Codex must:

1. inspect final diff;
2. run required checks;
3. verify no secrets/runtime artifacts;
4. commit with meaningful message;
5. push the work branch without force;
6. create/update handoff document;
7. report final branch + commit SHA.

## 15. Stop condition

After handoff, **do not begin the next roadmap task/phase** unless this task explicitly says otherwise. Architect review comes next.
