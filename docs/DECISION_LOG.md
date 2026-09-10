# Decision Log

Material architecture decisions are append-only entries. Superseding a decision requires a new entry that names the superseded ID; do not silently rewrite history.

---

## D-001 — Research-first, no automated execution

**Date:** 2026-09-10  
**Status:** ACCEPTED

### Decision

Initial product is read-only/paper-arbitrage research. It may discover, monitor, calculate, persist and alert, but it does not place Polymarket orders or sportsbook wagers.

### Why

The first unknown is whether opportunities are actually frequent, settlement-compatible, fresh and executable at useful size. Execution code would add risk and distract from validating the hypothesis.

### Consequence

No wallet/private key or sportsbook wagering automation belongs in MVP.

---

## D-002 — Windows-first modular monolith

**Date:** 2026-09-10  
**Status:** ACCEPTED

### Decision

Implement as one modular asynchronous Python application on the owner’s Windows workstation, with strict internal component boundaries.

### Why

Simplest deploy/debug model for one-machine research while preserving adapter/domain separation.

### Consequence

No microservice/message-broker architecture without measured need.

---

## D-003 — SQLite as MVP evidence store

**Date:** 2026-09-10  
**Status:** ACCEPTED

### Decision

Use SQLite with explicit migrations and WAL where appropriate for MVP.

### Why

Single-machine service, moderate expected watchlist and simple operations. Research evidence does not justify PostgreSQL before measurement.

### Consequence

PostgreSQL migration is an explicit future decision if contention/volume evidence requires it.

---

## D-004 — Discovery plane separated from hot watch

**Date:** 2026-09-10  
**Status:** ACCEPTED

### Decision

Large-universe discovery runs at a lower cadence. Low-latency monitoring applies only to already accepted matched markets in an active watchlist.

### Why

Avoid wasteful full-universe polling/recalculation and reduce storage/network load.

### Consequence

A quote update recalculates only affected matches.

---

## D-005 — Polymarket official public market data is primary Polymarket source

**Date:** 2026-09-10  
**Status:** ACCEPTED

### Decision

Use official Polymarket metadata/Gamma data for discovery and official CLOB REST/WebSocket market data for executable price/depth monitoring.

### Evidence

At bootstrap, official Polymarket documentation says public market data needs no API key/authentication/wallet and documents a public market WebSocket.

### Consequence

Third-party Polymarket quote aggregation is not required for MVP truth.

---

## D-006 — Fonbet acquisition remains PROBE_ONLY until measured

**Date:** 2026-09-10  
**Status:** ACCEPTED

### Decision

Do not assume a production Fonbet endpoint/transport. Phase 1 uses bounded headful Playwright/CDP observation to identify how the normally accessed page receives line/odds data.

### Why

Source transport, IDs, timestamps, stability and terms boundary are not yet proven on the owner’s machine.

### Consequence

Persistent Fonbet collector implementation begins only after architect review of probe evidence.

---

## D-007 — Settlement compatibility is a hard gate

**Date:** 2026-09-10  
**Status:** ACCEPTED

### Decision

Text/name similarity cannot make a market pair tradably equivalent. Required RuleSignature fields must be known and compatible.

### Consequence

Missing required semantics -> `UNVERIFIED`; conflict -> `REJECTED`. Confidence/fuzzy score cannot override.

---

## D-008 — Exclude unmodeled push/refund markets initially

**Date:** 2026-09-10  
**Status:** ACCEPTED

### Decision

A UI with two selections is not automatically a binary settlement contract. Integer totals/handicaps or other markets with an equality/push/refund state are excluded until a refund-aware cross-source model is designed and accepted.

### Consequence

Initial total/handicap work prioritizes non-push lines such as half-points where source settlement rules otherwise match.

---

## D-009 — Executable Polymarket asks/depth, not midpoint, drive paper arbitrage

**Date:** 2026-09-10  
**Status:** ACCEPTED

### Decision

For a hypothetical purchase of a complementary Polymarket outcome, use actual ask levels/depth. Midpoint, last trade and UI probability are diagnostics only.

### Consequence

Best-ask-only theoretical signals must pass depth/cost simulation before `VALIDATED_PAPER`.

---

## D-010 — Explicit fee/cost versioning

**Date:** 2026-09-10  
**Status:** ACCEPTED

### Decision

Polymarket/source costs are versioned configuration/data, not timeless constants.

### Evidence

Current Polymarket documentation states fees are market-dependent and provides per-market fee information/formula. Fees can change over time/category.

### Consequence

Research records identify cost-model version used; old runs are not silently recalculated as if current fees applied then.

---

## D-011 — Change-only normalized quote persistence

**Date:** 2026-09-10  
**Status:** ACCEPTED

### Decision

Persist material quote/state changes, not identical repeated poll results. Full/deeper order books are retained at resync/opportunity/evidence boundaries rather than every tick.

### Why

Reduces storage without sacrificing paper-opportunity reconstruction.

---

## D-012 — Repository-mediated architect/Codex workflow

**Date:** 2026-09-10  
**Status:** ACCEPTED

### Decision

Durable work coordination uses architect task docs, Codex handoffs and architect reviews in GitHub. `PROJECT_STATE.md` is the recovery cursor.

### Consequence

Codex does not independently move roadmap phases. Architect accepts/rejects work.

---

## D-013 — Repository-specific SSH deploy key for local Codex

**Date:** 2026-09-10  
**Status:** ACCEPTED

### Decision

Prefer a write-enabled deploy key scoped to `MaksimUnimax/Test_Poly` for local Codex Git access, with private key stored under the owner’s Windows `.ssh` directory.

### Consequence

Do not store SSH private key under `D:\Test_Poly` or use a broad PAT merely for this repository.

---

## D-014 — Main branch is architect-reviewed integration branch

**Date:** 2026-09-10  
**Status:** ACCEPTED

### Decision

Codex normally works on `codex/<phase>-<task>` branches and pushes them for review. No force pushes.

### Consequence

Direct Codex development on `main` is not default. Architect may merge/accept after review.

---

## Pending decisions

No pending architectural decision blocks Phase 1 tooling bootstrap.

The first expected material pending decision will be the **Fonbet persistent acquisition method**, after the Phase 1 browser/network probe.
