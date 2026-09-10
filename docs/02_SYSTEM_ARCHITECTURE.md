# 02 — System Architecture

## 1. Architectural style

A modular monolith for the MVP: one Python application with strict internal module boundaries and asynchronous I/O. This minimizes deployment complexity on Windows while preserving a clean path to split high-load components later if measurements justify it.

Do not introduce microservices, Kafka, Redis or PostgreSQL merely for architecture aesthetics. Scale decisions must follow measured need.

## 2. Major planes

### 2.1 Discovery plane

Purpose: find potentially comparable source markets.

Characteristics:

- relatively slow cadence;
- large universe allowed;
- metadata-heavy;
- tolerant of seconds/minutes of delay depending on prematch horizon;
- produces canonical source event/market candidates.

Components:

- `polymarket_discovery`;
- `sportsbook_discovery`;
- source parsers;
- normalizers;
- event matcher;
- market matcher;
- watchlist manager.

### 2.2 Hot path

Purpose: monitor only accepted matched markets and recalculate opportunities on relevant changes.

Characteristics:

- event driven where source supports push/WebSocket;
- bounded polling fallback where necessary;
- low allocation / no full-universe scans on each tick;
- freshness-aware;
- stateful current-quote cache backed by persisted evidence.

Components:

- source quote streams;
- quote state store/cache;
- per-match recalculation dispatcher;
- arbitrage engine;
- opportunity lifecycle manager;
- evidence snapshotter;
- alert policy;
- Telegram adapter.

### 2.3 Persistence/analysis plane

Purpose: preserve enough evidence to reproduce results without storing every byte forever.

Components:

- SQLite repositories;
- raw bounded diagnostic capture;
- normalized change history;
- opportunity evidence;
- replay reader;
- research exporter.

## 3. Proposed package layout

```text
backend/
  app/
    main.py
    config.py
    lifecycle.py

    domain/
      enums.py
      events.py
      markets.py
      quotes.py
      matching.py
      opportunities.py
      health.py

    collectors/
      base.py
      polymarket/
        discovery.py
        clob_rest.py
        market_ws.py
        parser.py
      fonbet/
        browser_probe.py
        collector.py        # only after probe establishes approved method
        parser.py
      third_party/          # empty until architect approves a provider

    normalization/
      participants.py
      competitions.py
      events.py
      markets.py
      rules.py

    matching/
      event_matcher.py
      market_matcher.py
      rule_compatibility.py
      watchlist.py

    pricing/
      quote_state.py
      fees.py
      depth.py
      arbitrage.py
      opportunity_lifecycle.py

    storage/
      db.py
      models.py
      repositories.py
      migrations/
      replay.py
      retention.py

    notifications/
      telegram.py
      policy.py

    observability/
      logging.py
      metrics.py
      health.py

    api/
      routes_health.py
      routes_debug.py
      routes_research.py

  tests/
    unit/
    contract/
    integration/
    replay/
    fixtures/

scripts/
  setup_windows.ps1
  run_scanner.ps1
  run_tests.ps1
  probe_fonbet.ps1
  export_research.ps1

config/
  scanner.example.yaml

data/                  # gitignored runtime tree
logs/                  # gitignored
browser_profiles/      # gitignored
```

Names may be adjusted for Python packaging conventions, but module responsibilities require architect approval to materially change.

## 4. Core interfaces

### 4.1 DiscoveryCollector

Conceptual contract:

```text
start()
stop()
discover() -> async stream/batch of SourceEvent + SourceMarket
health() -> SourceHealth
```

A collector owns source transport details but not canonical matching decisions.

### 4.2 QuoteCollector

```text
subscribe(source_market_refs)
unsubscribe(source_market_refs)
updates() -> async stream of SourceQuoteUpdate
resync(ref)
health()
```

Polling implementations may emulate subscriptions internally.

### 4.3 Normalizer

```text
normalize_event(source_event) -> CanonicalEventCandidate
normalize_market(source_market) -> CanonicalMarketCandidate
normalize_quote(source_quote) -> CanonicalQuote
```

Normalization must be deterministic for the same source payload/config version.

### 4.4 Matcher

```text
match_event(a, b) -> EventMatchDecision
match_market(a, b, event_match) -> MarketMatchDecision
```

Decision includes status, score where useful, hard gates, reason codes and rule-signature comparison.

### 4.5 ArbitrageEngine

```text
evaluate(match, quote_state, cost_model) -> OpportunityEvaluation
```

Pure domain calculation as far as practical: no network I/O and no Telegram/database side effects inside formula evaluation.

## 5. Message flow

### 5.1 Discovery

```text
source payload
  -> source parser
  -> source event/market records
  -> canonical normalizer
  -> candidate index
  -> event match
  -> market/rule match
  -> watchlist decision
```

### 5.2 Quote update

```text
source update
  -> parse/validate
  -> canonical quote update
  -> current quote state
  -> identify affected MarketMatch IDs
  -> arbitrage evaluation
  -> opportunity lifecycle transition
  -> persistence
  -> optional evidence snapshot
  -> optional alert
```

Only affected matches are evaluated.

## 6. Current-state cache

The service maintains in-memory current state keyed by stable source/canonical identifiers:

- last accepted quote/update per source leg;
- Polymarket best ask and required depth view;
- current source health;
- match/watchlist membership;
- active opportunity state.

SQLite remains the evidence system of record for research history, not the low-latency read path for every calculation.

On restart current quote state is considered stale until resynchronized.

## 7. Queue/backpressure strategy

Use bounded `asyncio.Queue`-style boundaries between transport parsing and domain processing where needed.

Rules:

1. control/lifecycle events (disconnect, suspend, close) must not be silently dropped;
2. source snapshots required for resync must not be replaced by deltas that depend on them;
3. for simple current-price updates, multiple queued superseded updates for the same source key may be coalesced if research semantics are preserved;
4. an update that opens/closes a recorded opportunity cannot be discarded from opportunity evidence after it has reached evaluation;
5. queue overflow is an observable health incident, not silent behavior.

Concrete capacities/coalescing thresholds are calibrated in Phase 1/2.

## 8. Source isolation

Each connector has:

- independent lifecycle;
- independent retry/backoff;
- independent rate policy;
- health state;
- parsing error metrics;
- source-specific raw diagnostics.

A Fonbet failure must cause related matches to become stale/unavailable, not stop Polymarket collection or corrupt Polymarket state.

## 9. Reconnect/resynchronization

For push feeds:

1. mark source/affected subscriptions `RESYNCING` on disconnect;
2. invalidate old quote freshness;
3. reconnect with bounded exponential backoff + jitter;
4. obtain a known-good snapshot where protocol provides one;
5. apply deltas only after snapshot baseline is valid;
6. return subscriptions to `HEALTHY`;
7. only then permit opportunity validation.

Never infer continuity across an unknown data gap.

## 10. Discovery vs hot-watch rates

Do not hard-code arbitrary universal polling rates.

The configuration model supports:

- discovery interval per source and mode;
- hot polling interval if no push feed exists;
- minimum/maximum backoff;
- staleness thresholds per source/mode.

Phase 1 measures how the source itself updates before choosing aggressive polling.

## 11. External API surface

A small local FastAPI surface is allowed for operations/debugging. Bind to loopback by default.

Candidate endpoints:

- `GET /health`;
- `GET /sources`;
- `GET /watchlist`;
- `GET /matches`;
- `GET /opportunities/active`;
- `GET /opportunities/{id}`;
- `POST /research/export` or CLI equivalent.

No remote unauthenticated control plane should be exposed by default.

## 12. Scaling path

Only if measurements show a real need:

1. SQLite -> PostgreSQL for concurrent/high-volume persistence;
2. process separation for browser collector from core scanner;
3. durable queue/message broker only if process decoupling and replay requirements justify it;
4. distributed deployment only after local architecture is proven.

No speculative scaling dependencies in MVP.

## 13. Dependency policy

Preferred dependency classes:

- Python standard library / asyncio;
- `httpx` or equivalent async HTTP client;
- `websockets`/official client where appropriate;
- Playwright Python for browser diagnostics/collector if approved by probe;
- FastAPI + Uvicorn for local diagnostics API;
- Pydantic v2 / pydantic-settings for contracts/config;
- SQLAlchemy 2 + Alembic, or a similarly explicit migration-capable persistence layer;
- pytest + pytest-asyncio.

A task may propose alternatives, but material dependency changes are architect decisions.
