# 08 — Test Strategy

## 1. Test philosophy

The dominant project risk is a **false opportunity** caused by bad parsing, wrong event/market matching, stale data, incorrect side orientation, fee mistakes or incomplete settlement rules.

Tests therefore prioritize semantic correctness and replayability over superficial endpoint coverage.

## 2. Test layers

### 2.1 Unit tests

Pure logic:

- participant/name normalization;
- time normalization;
- line parsing;
- market-family parsing;
- rule-signature construction;
- event candidate scoring;
- hard-gate decisions;
- outcome orientation;
- decimal-odds conversion helpers;
- depth consumption;
- arbitrage formulas;
- fee/cost formulas;
- staleness decisions;
- opportunity lifecycle transitions;
- alert deduplication.

### 2.2 Parser fixture tests

Every external source parser must have sanitized fixtures representing real observed payload classes.

Fixtures should include:

- valid normal case;
- missing optional fields;
- suspension;
- closure;
- line change;
- reordered participants/outcomes;
- malformed/unknown record;
- source schema variation observed in the wild.

Parser behavior must be fail-observable: unknown schema cannot silently produce a plausible wrong quote.

### 2.3 Contract tests

Assert that every collector emits domain contracts conforming to `docs/03_DATA_CONTRACTS.md`:

- timezone-aware UTC;
- `Decimal`-compatible numeric semantics;
- source IDs preserved;
- acquisition/parser provenance present;
- no source-specific numeric interpretation leaking into canonical semantics.

### 2.4 Matcher tests

Positive and negative matrices are mandatory.

Positive examples:

- participant order reversed but otherwise same event;
- accepted alias spelling;
- exact half-point total with same period/rules;
- exact two-way winner with compatible settlement rules.

Negative/unverified examples:

- same names, different start instance;
- first half vs full match;
- map 1 vs match;
- +1.5 vs -1.5 orientation error;
- 2.5 vs 3.5;
- regulation-only vs overtime-included;
- tennis retirement policy conflict;
- integer push-capable line with unmodeled refund;
- one required rule field unknown;
- fighter wins by KO vs fight ends by KO;
- 1/X/2 market presented as binary subset.

Every new market family must add family-specific negative cases before being enabled.

### 2.5 Arbitrage calculation tests

Use hand-verifiable examples.

Example no-fee condition:

```text
sportsbook decimal odds = 2.10
Polymarket complementary ask = 0.50
cost_ratio = 1/2.10 + 0.50
           ≈ 0.9761904762
payout_margin ≈ 0.0238095238
capital_roi = 1/cost_ratio - 1
```

Tests must verify exact `Decimal` behavior to configured precision and distinguish payout margin from ROI on capital.

Include boundary cases:

- equality `cost_ratio == 1` is not arbitrage;
- tiny apparent edge removed by fees;
- best ask passes but depth-weighted average fails;
- insufficient depth;
- stale one leg;
- suspended one leg;
- size unknown;
- fee version changes result.

### 2.6 Depth tests

Fixtures with multiple asks must verify:

- ascending consumption;
- partial level fill;
- exact exhaustion;
- insufficient book;
- weighted total cost;
- maximum size under minimum ROI threshold.

### 2.7 Opportunity lifecycle tests

State-machine cases:

```text
NONE -> THEORETICAL -> VALIDATED_PAPER -> EXPIRED
NONE -> THEORETICAL -> REJECTED_COSTS -> EXPIRED
VALIDATED_PAPER -> REJECTED_STALE -> EXPIRED
expired -> new episode after reappearance
```

Rapid alternating quotes must not generate duplicate lifecycle records or alert storms.

### 2.8 Async/reconnect tests

Use fake streams to test:

- WebSocket disconnect;
- resync snapshot requirement;
- delta before snapshot rejection;
- duplicate snapshot;
- duplicate sequence;
- out-of-order update policy;
- bounded reconnect backoff;
- graceful shutdown during reconnect;
- source failure isolation.

### 2.9 Backpressure tests

Generate bursts larger than queue capacity and verify the designed behavior:

- no unbounded memory growth;
- control events preserved;
- superseded current-state ticks coalesced only under documented rules;
- health incident emitted;
- opportunity transition evidence not silently lost.

### 2.10 Storage tests

- fresh database bootstrap;
- every migration forward path;
- foreign-key integrity;
- idempotent upsert semantics;
- quote-change deduplication;
- research-run provenance;
- retention avoids referenced opportunity evidence;
- safe backup/snapshot;
- replay export/import ordering.

### 2.11 Replay tests

A captured normalized fixture stream should produce a deterministic expected sequence of:

- quote states;
- opportunity evaluations;
- lifecycle transitions;
- alerts under a fixed policy.

Replay tests are the main regression barrier for changing matcher/calculator logic.

### 2.12 Telegram tests

Use a fake/mocked Telegram transport for CI/unit tests.

Verify:

- correct trigger policy;
- payload contains paper marker and opportunity ID;
- retries are bounded/idempotent;
- error does not crash scanner;
- no secret values are interpolated into logs;
- duplicate alert suppression.

Live Telegram send is an explicit integration probe, not a default test.

## 3. Live probe tests

Live tests are opt-in and clearly marked, e.g. a pytest marker or separate script.

Never make normal unit/CI tests depend on:

- Polymarket being reachable;
- Fonbet page being reachable;
- Telegram being reachable;
- owner credentials.

Each live probe records a manifest and result.

## 4. Polymarket acceptance probes

At minimum:

1. discovery can fetch and parse active sports metadata/events/markets;
2. selected CLOB token ID can obtain a valid order-book snapshot;
3. market WebSocket receives a snapshot/update/heartbeat as documented;
4. reconnect/resubscribe path works in controlled simulation and at least one bounded live observation when practical;
5. source IDs/timestamps are persisted correctly.

## 5. Fonbet Phase 1 probe acceptance

The probe is successful even if final result is `BLOCKED`, provided evidence is complete.

Possible outcomes:

- `PASS_STRUCTURED_PUSH`;
- `PASS_STRUCTURED_POLLING`;
- `PARTIAL_DOM_ONLY`;
- `AUTH_REQUIRED_REVIEW`;
- `BLOCKED_ACCESS_CONTROL`;
- `UNSUITABLE_DATA`;
- `TERMS_REVIEW_REQUIRED`.

Do not label the connector production-ready from one successful page parse.

## 6. Test data policy

Test fixtures must:

- contain no credentials/cookies;
- remove irrelevant personal/session identifiers;
- preserve enough original structure to test parsers;
- document source/date/schema class;
- be small enough for review;
- include expected normalized output where practical.

## 7. Quality tools

Initial intended gates:

- `pytest`;
- `ruff` for lint/format or architect-approved equivalent;
- `mypy` or `pyright` for meaningful type checking, selected during bootstrap;
- dependency/security scanner after lockfile/CI exists;
- secret scan.

The bootstrap task will pin exact commands in project configuration.

## 8. Coverage policy

A numeric coverage percentage is not sufficient as a release gate. Critical semantic paths listed above must be explicitly tested.

Coverage reporting may be used to identify gaps, but 100% line coverage with missing negative settlement cases is a failure.

## 9. Regression policy

When a production/live probe exposes a parsing/matching/calculation defect:

1. preserve a sanitized minimal fixture reproducing it;
2. add a failing regression test;
3. fix the defect;
4. run relevant suite + full suite;
5. record material semantic change in decision/state docs if required.

## 10. Performance characterization

Do not invent HFT targets.

Benchmark after real sample acquisition:

- parser messages/sec;
- normalization latency;
- evaluation latency;
- queue high-water marks;
- SQLite write throughput/latency;
- memory under representative watchlist sizes;
- reconnect/resync time.

Then set explicit SLOs if needed.

## 11. Phase acceptance test rule

Codex reports test evidence; the architect decides whether the phase gate passes. A green unit suite does not override failed live evidence or unresolved semantic uncertainty.
