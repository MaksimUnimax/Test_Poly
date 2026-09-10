# 12 — Acceptance Gates

## 1. Rule

Acceptance is evidence-based. “Code exists” and “works on my machine once” are not sufficient.

The architect applies the gates relevant to each task/phase. Any blocking gate failure means `REWORK_REQUIRED` or `BLOCKED_DECISION_REQUIRED`.

## GATE A — Authority compliance

PASS requires:

- implementation matches current authority documents;
- no unauthorized architecture/scope expansion;
- any material decision is recorded;
- task acceptance criteria were not rewritten by Codex;
- current project state is recoverable.

Automatic FAIL examples:

- automated betting code added in MVP;
- new source/provider introduced without approval;
- canonical market semantics changed without decision;
- access-control bypass added.

## GATE B — Git/repository hygiene

PASS requires:

- correct base/branch;
- no force push;
- diff contains only intended scope;
- no unrelated local changes destroyed;
- meaningful commit(s);
- pushed remote SHA available;
- clean or explicitly accounted working tree at handoff.

## GATE C — Secret/data hygiene

PASS requires zero committed:

- `.env` real values;
- API/bot tokens;
- wallet/private keys;
- SSH private keys;
- cookies/session databases;
- unsanitized HAR/network dumps;
- browser profiles;
- runtime databases/logs unless deliberately approved fixture/artifact.

If a secret is discovered in commit history, stop and treat as an incident.

## GATE D — Build/tooling quality

For implementation tasks, PASS requires all task-specified commands succeed:

- environment/bootstrap;
- formatting;
- lint;
- type checks;
- unit/contract/integration tests.

A tool may be temporarily absent only if the phase explicitly has not introduced it yet; handoff must state that clearly.

## GATE E — Data contract correctness

PASS requires:

- UTC-aware timestamps;
- `Decimal` semantics where money/odds/price require it;
- source IDs/provenance preserved;
- unknown values explicit rather than fabricated defaults;
- schema migration/replay compatibility where persistence changes;
- source-specific representation does not leak into calculation incorrectly.

## GATE F — Source connector correctness

For a source connector/probe, PASS requires:

- acquisition mode documented;
- bounded rate/retry behavior;
- suspension/closure handling;
- reconnect/resync semantics where relevant;
- malformed/unknown payloads fail observably;
- representative sanitized fixtures;
- source health state;
- no access bypass;
- live verification if task requires it.

A parser based on one happy-path payload is not production-ready.

## GATE G — Matching semantic safety

PASS requires:

- event hard gates implemented/tested;
- exact market family and line mapping;
- period/scope equivalence;
- participant/side orientation correct;
- required settlement-rule fields known and compatible;
- incompatible rules -> `REJECTED`;
- missing required rules -> `UNVERIFIED`;
- no fuzzy text score can override hard incompatibility;
- positive + negative regression matrix passes.

Any demonstrated false match is a blocking defect.

## GATE H — Arbitrage mathematics

PASS requires:

- complementary outcomes only;
- executable Polymarket side used;
- no-fee condition correct;
- `payout_margin` and `capital_roi` not conflated;
- depth consumption correct;
- current fee/cost model version applied;
- equality/boundary cases tested;
- stale/suspended legs cannot validate;
- unknown sportsbook maximum not invented;
- all calculations use deterministic numeric precision policy.

Any false-positive profit calculation is a blocking defect.

## GATE I — Opportunity lifecycle and alerts

PASS requires:

- deterministic open/validate/reject/expire transitions;
- no duplicate episodes from duplicate ticks;
- alert deduplication;
- evidence exists for alert;
- Telegram failure is isolated/retry-bounded;
- alert clearly identifies `paper` status;
- no secret data in notification.

## GATE J — Async/reliability

PASS requires tests/evidence for applicable behavior:

- graceful shutdown;
- reconnect;
- resync after data gaps;
- old quotes invalid after restart;
- bounded queues/backpressure;
- source failure isolation;
- idempotent duplicate handling;
- no silent use of stale cached values.

## GATE K — Storage/replay

PASS requires:

- clean DB bootstrap/migrations;
- write/read integrity;
- retention policy does not orphan retained evidence;
- replay does not contact live providers;
- deterministic replay produces expected state/opportunities;
- research run stores version/config provenance.

## GATE L — Windows operability

For runnable milestones, PASS requires on the target Windows machine:

- documented setup works from clean clone or clearly specified prerequisites;
- PowerShell commands/scripts work;
- paths do not depend on a developer-specific home directory;
- browser dependencies are reproducible if needed;
- default local API binding is loopback;
- Ctrl+C/graceful stop works;
- runtime files stay outside Git.

## GATE M — Live probe evidence

When a task requires a live probe, PASS requires a manifest containing:

- source;
- purpose;
- UTC interval;
- code/commit version;
- config fingerprint without secrets;
- request/message counts where practical;
- result classification;
- errors/limitations;
- sanitized fixture/evidence path.

If no live probe was run, Codex must say so; it cannot infer PASS.

## GATE N — Research validity

For Phase 8 observation campaign, PASS requires:

- stable enough source uptime to interpret results;
- matching funnel reported;
- source freshness distribution reported;
- theoretical and validated counts separated;
- costs/depth/staleness applied;
- opportunity lifetimes measured;
- size-known vs size-unknown distinguished;
- source/matcher/calculator versions frozen per run or changes clearly segmented;
- incidents disclosed;
- conclusions do not exceed evidence.

A finding of “no meaningful arbitrage” can PASS this gate if measurement quality is sufficient.

## GATE O — Documentation/handoff

PASS requires:

- task handoff complete;
- exact files/behavior changed documented;
- exact tests/results documented;
- live status distinguished from mocked/tested status;
- unresolved issues stated;
- pushed commit SHA stated;
- no misleading “done” claim beyond verified evidence.

## Severity levels for architect review

### BLOCKER

Can create wrong market match, wrong opportunity, data corruption, credential exposure, unauthorized source behavior or inability to reproduce results. Must fix before acceptance.

### MAJOR

Material reliability/maintainability defect likely to invalidate a research run or make operation unsafe. Normally must fix before phase acceptance.

### MINOR

Non-blocking defect/documentation/ergonomics issue that can be deferred with an explicit record.

## Phase acceptance rule

A phase is `ACCEPTED` only after:

1. all required task handoffs exist;
2. relevant gates pass;
3. architect review is complete;
4. `PROJECT_STATE.md` is updated by/under architect instruction.

Codex implementation completion alone does not transition the roadmap phase.
