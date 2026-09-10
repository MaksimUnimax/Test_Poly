# 01 — Product Specification

## 1. Product definition

A local Windows service that discovers, matches, monitors and measures cross-platform sports-market discrepancies between Polymarket and sportsbooks, starting with Fonbet.

The initial product is a **research / paper-arbitrage scanner**, not a betting bot.

## 2. User-visible behavior

The owner must be able to:

- start/stop the scanner locally;
- see source health and current acquisition mode;
- see discovered events and markets;
- see match candidates and why each was accepted/rejected/unverified;
- see the active watchlist;
- see live/prematch quote freshness;
- receive Telegram alerts only for opportunities that pass configured gates;
- inspect evidence for an alert after the fact;
- replay captured normalized data without contacting external sources;
- export summary statistics for a research window.

A web UI is optional after core backend correctness. CLI/API diagnostics are sufficient for early phases.

## 3. Functional requirements

### FR-001 — Source health

For every configured source the system shall maintain:

- state: `DISABLED | STARTING | HEALTHY | DEGRADED | BLOCKED | FAILED`;
- last successful discovery/update UTC time;
- last error class without secrets;
- reconnect count;
- observed update rate;
- current data-age metrics.

### FR-002 — Polymarket discovery

The system shall discover active sports events/markets using public Polymarket metadata endpoints, storing source IDs and enough metadata to derive CLOB token IDs and market rule signatures.

### FR-003 — Polymarket hot feed

The system shall subscribe only to token IDs required by the active watchlist where possible. It shall process order-book snapshots and incremental price/depth updates and survive reconnects without silently using stale state.

### FR-004 — Sportsbook discovery

A sportsbook collector shall enumerate relevant events/markets using the acquisition method approved for that source. The first Fonbet phase is a probe: determine whether useful data arrives through public browser page state, XHR/fetch, WebSocket/push, or another approved method.

### FR-005 — Discovery filters

Configurable filters shall limit the candidate universe by:

- source;
- sport;
- league/competition allow/deny list;
- prematch/live state;
- event start horizon;
- market-family whitelist;
- settlement compatibility policy.

### FR-006 — Canonical normalization

Each source record shall be transformed into source-independent canonical entities while preserving provenance and raw source identifiers.

### FR-007 — Event matching

The matcher shall identify candidate same-event pairs using normalized participants, competition, start time and sport-specific identifiers/rules.

It shall assign a confidence score and reason codes, but confidence alone cannot override hard incompatibility.

### FR-008 — Market matching

A market pair can become `MATCHED` only if the required rule signature for its market family is equal/compatible.

Required dimensions include, as applicable:

- market family;
- outcome/side semantics;
- numeric line;
- period/scope;
- match/map/set/round identity;
- overtime/extratime treatment;
- push/refund possibility;
- retirement/withdrawal rules;
- void/cancellation behavior;
- participant orientation.

### FR-009 — Explicit match lifecycle

A candidate pair shall have one of:

- `CANDIDATE`;
- `UNVERIFIED`;
- `MATCHED`;
- `REJECTED`;
- `SUSPENDED`;
- `CLOSED`.

State changes shall be auditable by reason code.

### FR-010 — Active watchlist

Only accepted matched markets enter the hot watchlist. Discovery continues independently and adds/removes watch items as markets appear, start, suspend or close.

### FR-011 — Change-driven quote state

The hot path shall update only affected canonical quote state when a source update arrives. It must not rescan/recompute the entire universe on each tick.

### FR-012 — Staleness gate

No opportunity can be `VALIDATED` or alerted if either source quote exceeds its configured age threshold.

Thresholds shall be configurable by source and mode (prematch/live) and must not be hard-coded into business logic.

### FR-013 — Executable Polymarket side

For a hypothetical Polymarket purchase, calculations shall use the relevant ask levels and available sizes. Midpoint/displayed probability is informational only.

If a future strategy needs to sell an existing position, that is a separate trade model and is out of MVP unless explicitly added.

### FR-014 — Sportsbook executable price

The system shall store the displayed decimal odds and, where observable without account automation, source maximum/availability information. If executable maximum is unknown, opportunity size shall be labeled sportsbook-size-unverified rather than assumed.

### FR-015 — Cost model

The calculator shall support configurable source costs, including:

- Polymarket fees applicable to the market/trade model;
- configurable safety slippage reserve;
- optional fixed/percentage research cost adjustments;
- zero/unknown fee states explicitly represented.

Costs shall be versioned/configured, not buried as constants.

### FR-016 — Opportunity states

At minimum:

- `THEORETICAL` — price inequality passes before all gates;
- `REJECTED_STALE`;
- `REJECTED_RULES`;
- `REJECTED_DEPTH`;
- `REJECTED_COSTS`;
- `SIZE_UNVERIFIED`;
- `VALIDATED_PAPER`;
- `EXPIRED`.

### FR-017 — Opportunity evidence

When a theoretical opportunity first appears, persist:

- both source quotes;
- all relevant timestamps;
- matched event/market IDs;
- market rule signatures;
- Polymarket top-of-book and enough depth for sizing;
- modeled costs;
- theoretical and validated edge;
- rejection/validation reason;
- configured thresholds/version identifiers.

For validated opportunities, take/store a deeper order-book snapshot when practical.

### FR-018 — Opportunity lifetime

The system shall track:

- first seen;
- first validated;
- last continuously valid;
- expired time;
- maximum edge;
- maximum validated executable size;
- number of quote changes during lifetime.

### FR-019 — Telegram alerts

Telegram alerts shall be sent only for opportunities that pass configurable alert policy, for example:

- `VALIDATED_PAPER`;
- minimum post-cost edge;
- minimum freshness;
- minimum Polymarket executable size;
- cooldown/deduplication policy;
- optional sportsbook-size verification status.

Alert messages shall include enough identifiers to find the local evidence record.

### FR-020 — Alert deduplication

The system must not spam repeated alerts for unchanged conditions. Re-alert requires a configured material change, expiry-and-reappearance, or cooldown policy.

### FR-021 — Persistent storage

MVP shall persist normalized entities, match decisions, quote changes needed for research, opportunity records, alerts and source-health incidents in SQLite.

### FR-022 — Raw diagnostic capture

Raw source payload capture is optional in normal operation and enabled for bounded diagnostics. It shall have retention limits and redaction rules.

### FR-023 — Replay mode

The scanner shall be able to feed stored normalized quote events through matching/arbitrage logic without external network calls, enabling deterministic regression tests and research reproduction.

### FR-024 — Research report/export

For a selected UTC interval, produce at least CSV/JSON summary data containing:

- discovered events by source/sport;
- discovered markets by family;
- candidate/matched/rejected/unverified counts;
- rejection reasons;
- quote count/update rates;
- source age percentiles;
- theoretical opportunities;
- validated paper opportunities;
- opportunity lifetime statistics;
- edge statistics;
- size statistics;
- source health incidents.

### FR-025 — Configuration

Configuration shall cover:

- enabled sources;
- allowed sports/families;
- discovery horizon;
- polling intervals when needed;
- stale thresholds;
- matching tolerances;
- edge/size alert thresholds;
- Telegram credentials via environment variables;
- storage paths;
- diagnostic raw-capture switch and retention.

Configuration validation shall fail fast with useful errors.

### FR-026 — Graceful shutdown

Shutdown shall stop new work, flush pending database writes, close WebSockets/browser contexts and leave enough state for clean restart.

### FR-027 — Restart recovery

After restart, the application shall not assume cached quotes remain fresh. It shall rebuild/resynchronize hot state before validating new opportunities.

### FR-028 — Time handling

External/domain timestamps shall be stored in UTC. Latency measurement shall distinguish source timestamp, receive timestamp and processing timestamp where available.

### FR-029 — Clock sanity

Startup diagnostics shall warn if local wall-clock behavior appears inconsistent with source timestamps beyond a configured tolerance. Duration measurement inside a process should use a monotonic clock where appropriate.

### FR-030 — Error isolation

Failure of one source collector shall not corrupt canonical state for another source. Source state becomes degraded/failed and related opportunities are invalidated as stale rather than silently continuing.

## 4. Non-functional requirements

### NFR-001 — Correctness over throughput

False arbitrage caused by bad matching, stale quotes or incorrect settlement semantics is a release blocker.

### NFR-002 — Local Windows operation

The MVP must run natively on a Windows workstation without requiring Linux or Docker.

### NFR-003 — Async/event-driven hot path

The backend shall use asynchronous I/O and bounded queues. Hot-path recomputation is keyed to affected matched markets.

### NFR-004 — Backpressure

High update rates must not cause unbounded in-memory queues. The implementation shall define coalescing/drop policy for superseded quote-state updates while preserving opportunity evidence semantics.

### NFR-005 — Idempotency

Reconnects, repeated snapshots and duplicate source updates must not create duplicate canonical entities/opportunities/alerts.

### NFR-006 — Observability

Structured logs and health metrics must make it possible to distinguish source outage, parser failure, matcher rejection, stale state, calculator rejection and Telegram failure.

### NFR-007 — Security

No secrets, cookies or authenticated raw headers in Git or normal logs. Secret values are environment/runtime-only.

### NFR-008 — Testability

Every external collector shall have fixture/replay seams so tests can run without live providers.

### NFR-009 — Extensibility

Adding a sportsbook must primarily require a new adapter plus source-specific parsing/rule mapping, not changes to the arbitrage core.

### NFR-010 — Performance target

The initial architecture should comfortably handle hundreds to low thousands of actively watched markets on a normal desktop without treating this as a high-frequency-trading system. Concrete latency/throughput targets will be calibrated from Phase 1 measurements rather than invented in advance.

## 5. Initial alert payload

A Telegram alert should eventually contain fields equivalent to:

- sport / event;
- market and line;
- source A side + executable price;
- source B side + decimal odds;
- post-cost theoretical edge;
- validated executable size or explicit `SIZE_UNVERIFIED`;
- age of each leg;
- opportunity ID;
- first-seen UTC and current lifetime;
- paper-only marker.

No credential/account information may appear.

## 6. Product acceptance boundary

An end-to-end demo that compares two hard-coded prices is not the MVP. MVP acceptance requires real source acquisition, canonical matching, freshness controls, persisted evidence, deterministic replay tests and a paper observation run that produces measurable research output.
