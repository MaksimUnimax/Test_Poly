# 06 — Storage, Retention and Observability

## 1. Storage objectives

The MVP needs enough evidence to reproduce and audit a paper opportunity without archiving every byte of every source forever.

Storage policy follows three layers:

1. **bounded raw diagnostics** — short-lived source payload evidence used to build/fix collectors;
2. **normalized change history** — compact domain events used for replay/research;
3. **opportunity evidence** — durable records for theoretical/validated episodes, snapshots and alerts.

## 2. MVP database

Use SQLite with:

- WAL mode where appropriate;
- explicit migrations;
- foreign keys enabled;
- transactional writes;
- indexes designed from actual query paths;
- database file under gitignored `data/`.

SQLite is a deliberate MVP choice. Migration to PostgreSQL requires measured evidence of contention/volume that SQLite cannot safely handle.

## 3. Suggested logical tables

Names are illustrative but responsibilities are authoritative:

```text
source_events
source_markets
canonical_event_candidates
event_match_decisions
canonical_market_candidates
market_match_decisions
quote_changes
orderbook_snapshots
cost_model_versions
opportunity_evaluations
opportunity_lifecycles
alert_records
source_health_incidents
research_runs
schema_metadata
```

Raw diagnostic payload files may remain on disk with metadata/index rows rather than as database BLOBs.

## 4. Change-only quote persistence

Do not persist repeated identical values just because a poll occurred.

Persist a normalized quote change when one or more material fields change:

- price/odds;
- available size/depth bucket used by research;
- open/suspended/closed state;
- line/rule identity;
- source sequence/version where needed for replay integrity.

Periodic heartbeat/health sampling is separate from quote history.

Example:

```text
12:00:01.100 ask=0.55 size=120
12:00:01.400 ask=0.55 size=120    -> not another quote-change row
12:00:08.222 ask=0.54 size=95     -> persist
```

## 5. Order-book persistence

Normal hot state may keep only the depth needed for configured sizing.

Persist a full/deeper Polymarket snapshot when:

- WebSocket/REST resynchronization requires it;
- an opportunity first crosses theoretical threshold;
- an opportunity becomes `VALIDATED_PAPER`;
- maximum edge/size materially changes under evidence policy;
- an explicit bounded diagnostic is running.

Do not save a full book on every insignificant tick unless a research task explicitly requests it.

## 6. Raw diagnostic storage

Suggested runtime tree:

```text
data/raw/<source>/<run_id>/
  manifest.json
  *.jsonl
  summary.md
```

Every run manifest includes:

- run ID;
- source;
- acquisition method;
- start/end UTC;
- collector/parser version;
- purpose;
- configuration fingerprint with secrets removed;
- record counts;
- redaction version;
- result status.

Raw files are never committed directly from a browser/account session. Sanitized minimal fixtures may be deliberately copied into `tests/fixtures/` after inspection.

## 7. Initial retention defaults

Defaults are configuration, not hard-coded domain truth:

```text
raw diagnostic captures:        24 hours
normal logs:                     14 days
normalized quote changes:       14 days hot
source health samples:           30 days
match decisions:                 keep for research project
opportunity lifecycles/evidence: keep for research project
alerts:                          keep for research project
research exports:                keep until owner deletes
```

Before automatic deletion, opportunity evidence referenced by a retained research run must not be orphaned.

For longer research windows, export/compress normalized events before hot retention deletes them.

## 8. Research run

Every controlled observation period should have a `ResearchRun` record:

```text
run_id
started_at_utc
ended_at_utc?
git_commit_sha
config_fingerprint
schema_version
collector_versions
matcher_version
calculator_version
cost_model_ids
sources
sports/market-family scope
status
notes
```

This makes results reproducible and prevents mixing runs from different logic versions without knowing it.

## 9. Replay format

Normalized replay events should be exportable as ordered JSONL (optionally compressed later) containing:

```text
event_time_utc
receive_time_utc
source
entity key
event type
normalized payload
schema/version
```

Replay order must be deterministic. When timestamps tie, use persisted sequence/insertion ordering.

Replay mode performs no external source requests.

## 10. Structured logging

Use structured logs. Every log event should include, where relevant:

```text
timestamp_utc
level
component
source?
run_id?
event_id?
market_id?
match_id?
opportunity_id?
event_code
message
```

Normal logs must avoid raw response bodies, cookies, authorization headers, tokens and browser-storage content.

Human-readable console rendering may be layered over structured logging, but machine-parseable event codes remain.

## 11. Required log event classes

At minimum:

```text
APP_START / APP_STOP
SOURCE_START / SOURCE_HEALTH_CHANGE
SOURCE_RECONNECT / SOURCE_RESYNC
SOURCE_RATE_LIMIT
SOURCE_BLOCKED
PARSER_ERROR
NORMALIZATION_ERROR
MATCH_DECISION_CHANGED
WATCHLIST_ADDED / WATCHLIST_REMOVED
QUEUE_BACKPRESSURE
QUOTE_STALE
OPPORTUNITY_OPENED
OPPORTUNITY_VALIDATED
OPPORTUNITY_REJECTED
OPPORTUNITY_EXPIRED
EVIDENCE_SNAPSHOT
ALERT_SENT / ALERT_FAILED
DB_ERROR
CLOCK_ANOMALY
RETENTION_DELETE
```

## 12. Health metrics

Expose/log current metrics such as:

- source state;
- seconds since last successful source update;
- received updates/sec by source;
- parsed/failed updates;
- active event/market/watchlist counts;
- queue depth/high-water mark;
- WebSocket reconnect count;
- current stale quote count;
- candidate/matched/unverified/rejected counts;
- current theoretical/validated opportunity counts;
- Telegram failures;
- SQLite write latency / write errors;
- process memory and event-loop lag if practical.

Do not add a heavyweight monitoring stack before these metrics are useful locally.

## 13. Research export

For an interval/run ID, generate machine-readable outputs under a gitignored/export directory:

```text
summary.json
events.csv
market_matching.csv
opportunities.csv
source_health.csv
latency_stats.json
README.md
```

The export README records definitions of metrics (`payout_margin`, `capital_roi`, freshness, size status) to prevent interpretation errors.

## 14. Backup and corruption strategy

For research runs longer than a few hours:

- checkpoint/backup SQLite safely using SQLite backup semantics rather than copying a live DB blindly;
- create an end-of-run DB snapshot or export;
- record hashes for exported research artifacts when used for conclusions.

Database corruption or migration failure is a test/release blocker.

## 15. Disk-pressure behavior

The scanner must not fill the workstation disk silently.

Implement configurable thresholds:

- warning threshold;
- critical threshold;
- stop raw capture first;
- preserve core opportunity evidence;
- degrade/stop collection cleanly if safe persistence can no longer be guaranteed.

Never delete opportunity evidence silently to keep raw debug traffic.
