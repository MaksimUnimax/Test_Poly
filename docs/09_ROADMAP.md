# 09 — Engineering Roadmap

## Rule

Phases are gated. Codex does not begin the next phase merely because code for the current phase exists. The architect reviews evidence and updates `PROJECT_STATE.md`.

No phase may introduce automated wagering under this roadmap.

---

# Phase 0 — Authority and repository bootstrap

## Goal

Freeze project purpose, architecture, data semantics, work protocol and acceptance rules before implementation.

## Deliverables

- README;
- AGENTS authority;
- charter/specification;
- system architecture;
- canonical data contracts;
- source strategy;
- matching/arbitrage model;
- storage/observability policy;
- security/compliance boundaries;
- test strategy;
- roadmap;
- Windows operations plan;
- Codex work protocol;
- acceptance gates;
- project state/decision log/references.

## Exit gate

Architect verifies all authority files exist and are internally consistent.

**Current state:** in progress at initial repository bootstrap.

---

# Phase 1 — Windows engineering bootstrap + bounded source probes

## Goal

Create a reproducible local development environment and measure the actual source interfaces before implementing business logic around assumptions.

## Work packages

### P1.1 Repository/tooling bootstrap

- Python project/package skeleton;
- virtual environment/bootstrap PowerShell script;
- dependency lock strategy;
- config example;
- `.gitignore` runtime/security rules;
- pytest + lint + type-check baseline;
- basic CI if practical for source-independent tests;
- application version/build metadata.

### P1.2 Polymarket public-data probe

Bounded script/probe proving:

- sports/event/market discovery;
- token IDs;
- CLOB book access;
- market WebSocket subscription;
- observed message types/timestamps;
- reconnect/resync behavior sufficient to design connector.

Persist a sanitized probe summary/fixtures.

### P1.3 Fonbet browser/network probe

Headful Playwright on owner machine, read-only.

Determine:

- normal accessibility;
- prematch/live page structure;
- XHR/fetch/WebSocket use;
- event/market/selection IDs;
- odds/suspension representation;
- timestamp/sequence behavior;
- approximate normal update rate;
- whether structured data is sufficient;
- terms/access blockers.

Do not build an evasion workaround if blocked.

### P1.4 Source architecture decision

Architect reviews probe evidence and records chosen Fonbet acquisition method in `DECISION_LOG.md`.

## Exit gate

- local setup reproducible;
- source probe evidence complete;
- no secrets committed;
- Polymarket acquisition path proven;
- Fonbet result classified (`structured push`, `structured polling`, `DOM fallback`, `blocked/review`, etc.);
- next collector architecture approved.

---

# Phase 2 — Canonical domain core and persistence

## Goal

Implement source-independent contracts before full source collectors.

## Work packages

### P2.1 Domain models

Implement enums/entities from `03_DATA_CONTRACTS.md` with Decimal/time semantics.

### P2.2 Configuration

Typed configuration, fail-fast validation, environment secret handling.

### P2.3 Persistence

SQLite schema/migrations/repositories for:

- source events/markets;
- canonical candidates;
- match decisions;
- quote changes;
- opportunity lifecycle/evaluations;
- health incidents;
- research runs;
- alerts.

### P2.4 Replay infrastructure

Normalized JSONL/event replay path with no external I/O.

### P2.5 Observability base

Structured logging, source/component event codes, local `/health` skeleton.

## Exit gate

- migrations pass clean/fresh DB tests;
- domain contract tests pass;
- replay round-trip deterministic;
- no external source dependency required for test suite.

---

# Phase 3 — Polymarket production-quality read-only connector

## Goal

Implement the first stable source end-to-end.

## Work packages

### P3.1 Discovery connector

Pagination/filtering/sports events/markets/token identifiers.

### P3.2 CLOB snapshot client

Book snapshot and explicit error/rate/backoff behavior.

### P3.3 Market WebSocket client

Subscription management, snapshot/deltas, heartbeat, reconnect, resync and health.

### P3.4 Polymarket normalizer

Event/market/RuleSignature/quote mapping.

### P3.5 Fixtures/replay

Sanitized real payload classes and deterministic tests.

## Exit gate

A bounded live run can discover configured sports markets, subscribe to selected tokens, persist normalized changes and recover from controlled reconnect without validating stale state.

---

# Phase 4 — Fonbet read-only collector

## Goal

Implement only the acquisition method approved after Phase 1 evidence.

Possible implementations:

- browser WebSocket observer;
- browser XHR/poll observer;
- documented/public endpoint client if discovered and approved;
- DOM fallback if structured data is genuinely unavailable and reliability is acceptable.

## Work packages

### P4.1 Transport/lifecycle

Source start/stop/health, bounded polling or browser lifecycle.

### P4.2 Parser

Event/market/selection/odds/suspension parsing.

### P4.3 Normalizer/rule extraction

Canonical candidates + rule signature provenance.

### P4.4 Fixtures/regressions

Multiple real sanitized payload states.

### P4.5 Bounded live stability run

Measure updates, parser errors, CPU/memory, access stability and data age.

## Exit gate

- approved read-only method only;
- stable IDs/data fields documented;
- no source control bypass;
- parser fixture matrix passes;
- bounded live run demonstrates usable data or the phase is honestly classified blocked/unsuitable.

If blocked/unsuitable, the architect decides whether to evaluate a third-party control/provider instead of inventing a workaround.

---

# Phase 5 — Event/market matching and watchlist

## Goal

Turn two source universes into a small safe set of settlement-compatible matched markets.

## Work packages

### P5.1 Participant/competition aliases

Deterministic normalization and explicit alias data.

### P5.2 Event matcher

Hard gates + explainable confidence/reasons.

### P5.3 Initial market families

Enable only approved families with family-specific RuleSignature checks.

Recommended initial order based on source overlap evidence:

1. simplest two-way winner family with compatible no-draw/settlement rules;
2. half-point totals;
3. BTTS;
4. selected half-point handicaps;
5. explicitly defined binary propositions.

The exact sequence can change after Phase 1 source evidence.

### P5.4 Match decision persistence

Candidate / unverified / matched / rejected audit trail.

### P5.5 Watchlist manager

Subscribe only matched open markets; remove on suspension/closure/invalidation.

## Exit gate

- required negative semantic tests pass;
- sampled accepted matches are manually auditable from persisted evidence;
- unknown settlement fields cannot become matched;
- watchlist is materially smaller than source universe and updates dynamically.

---

# Phase 6 — Arbitrage engine and evidence lifecycle

## Goal

Measure theoretical and validated paper opportunities correctly.

## Work packages

### P6.1 Current quote state

Fast in-memory keyed state with freshness.

### P6.2 Cost model

Versioned Polymarket/current costs and configurable safety reserve.

### P6.3 Depth-aware calculator

Both complementary directions; Decimal math; weighted Polymarket depth.

### P6.4 Staleness/source-health gates

No stale/resyncing source can validate opportunity.

### P6.5 Opportunity state machine

Open/validate/reject/expire, maximum edge/size/lifetime.

### P6.6 Evidence snapshotter

Preserve both legs, rule signatures, config/model versions and deeper order book when appropriate.

## Exit gate

Replay fixtures reproduce expected opportunities exactly; no-fee/fee/depth/stale boundary tests pass; live system records paper opportunities with auditable evidence.

---

# Phase 7 — Telegram and operator diagnostics

## Goal

Surface only high-quality paper opportunities without spam.

## Work packages

### P7.1 Alert policy

Minimum post-cost ROI, size/freshness and status rules.

### P7.2 Telegram adapter

Idempotent/retry-safe send path with mocked tests and one explicit live integration probe.

### P7.3 Local diagnostics

Health/source/watchlist/current-opportunity endpoints or CLI views.

## Exit gate

A validated replay/live paper opportunity generates exactly the intended alert, duplicates are suppressed, and Telegram failure cannot corrupt scanner state.

---

# Phase 8 — Controlled observation campaign

## Goal

Answer whether the product hypothesis is real.

## Run design

Start with a controlled prematch window. Add live only after prematch correctness and source latency are understood.

Suggested evidence sequence:

1. 2–4 hour engineering soak;
2. one full active sports session/day;
3. multi-day run;
4. target 7–14 day representative research window if the preceding runs are stable.

The architect may change duration based on evidence; duration is not a hard product constant.

## Required report

- uptime by source;
- discovered event/market counts;
- matching funnel;
- accepted/unverified/rejected reasons;
- quote/update freshness distributions;
- theoretical opportunities;
- post-cost/depth/freshness validated opportunities;
- lifetime distribution;
- executable Polymarket size distribution;
- sportsbook size-known/unknown distribution;
- false-positive investigations;
- incidents;
- conclusion: continue / revise / stop.

## Exit gate

Evidence is sufficient to decide product viability without relying on screenshots/marketing claims from third parties.

---

# Phase 9 — Expansion decision

Only after Phase 8.

Possible architect-approved directions:

- additional sportsbooks as comparison/control feeds;
- third-party odds provider benchmarking;
- additional market families;
- live-event tuning;
- PostgreSQL/process split if measured need exists;
- richer local UI;
- capital-allocation research;
- separate legal/technical design for manual-assisted or execution features.

None of these are automatically authorized by completing the MVP.

---

# Cross-phase mandatory gates

Every phase/task:

- starts from verified branch/HEAD;
- respects `AGENTS.md`;
- adds/updates tests for behavior changed;
- does not commit runtime secrets/data;
- reports exact test commands/results;
- commits/pushes atomically;
- is reviewed against `12_ACCEPTANCE_GATES.md`;
- updates `PROJECT_STATE.md` only under architect/task instruction.
