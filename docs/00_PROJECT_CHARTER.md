# 00 — Project Charter

## 1. Project name

**Test_Poly — Polymarket ↔ Sportsbook Research Scanner**

## 2. Mission

Create a Windows-first research scanner that can determine, with reproducible evidence, whether executable price discrepancies exist between Polymarket sports markets and sportsbook markets, beginning with Fonbet.

The scanner is not judged by the number of visually attractive alerts. It is judged by whether it can prove that both legs represent the same settlement proposition, were simultaneously available, were fresh enough to act on, and had enough executable size to support a real paper-arbitrage opportunity after modeled costs.

## 3. Project roles

### Architect / technical lead

Owns architecture, requirements, roadmap, interfaces, data contracts, technical decisions, review and acceptance. Architectural changes require explicit architect approval and an update to the corresponding authority document or decision log.

### Codex implementation agent

Implements assigned tasks, runs tests and approved diagnostics, commits and pushes evidence. Codex is not authorized to redefine scope or architecture.

### Owner/operator

Controls the Windows machine, GitHub repository, credentials, browser/account access and final operational use.

## 4. Primary research questions

The system must eventually answer quantitatively:

1. How many sports events are simultaneously represented on both sources?
2. How many apparently similar markets are actually settlement-compatible?
3. How quickly do prices on each source change and how stale can each feed become?
4. How often does a theoretical cross-platform arbitrage condition occur?
5. How many theoretical opportunities survive fee, liquidity, freshness and rule-equivalence checks?
6. What is the distribution of opportunity lifetime?
7. What executable notional is available at each opportunity?
8. Which sports / market families produce the most reliable matches?
9. Is Fonbet a uniquely useful counterparty source, or does Polymarket diverge similarly from broader sportsbook consensus?
10. Is continued development economically/technically justified based on measured evidence?

## 5. MVP outcome

The MVP must provide:

- Polymarket public market discovery;
- Polymarket real-time quote/order-book monitoring;
- a read-only Fonbet/source acquisition path proven by a diagnostic probe;
- canonical event/market/quote models;
- deterministic candidate matching plus explicit settlement-compatibility gating;
- active watchlist management;
- event-driven cross-platform calculation;
- paper opportunity records with timestamps, source freshness and available size;
- Telegram alerts for opportunities that pass configured gates;
- local storage and replayable evidence;
- health/diagnostic reporting;
- automated tests.

## 6. Explicit non-goals of MVP

The MVP does not:

- place bets or orders;
- log into accounts for the purpose of wagering;
- bypass geoblocks or access controls;
- solve CAPTCHA or evade anti-bot systems;
- promise guaranteed profit;
- support 1/X/2 markets;
- infer settlement equivalence purely from market names;
- treat displayed probability/midpoint as an executable Polymarket price;
- continuously archive the entire source universe when a filtered watchlist is sufficient;
- optimize capital allocation across many simultaneous opportunities beyond simple opportunity sizing.

## 7. Safety/quality principles

### 7.1 Evidence before automation

No execution-oriented feature is considered before the research scanner has generated a meaningful paper dataset and the architect accepts the evidence.

### 7.2 False positive is worse than missed opportunity

The matcher and calculator should prefer `UNVERIFIED` / `REJECTED` over a false `MATCHED` state.

### 7.3 Settlement rules are data

Overtime, retirement, cancellation, void/refund and push behavior are part of market identity.

### 7.4 Source freshness is first-class

An attractive price comparison is invalid if either leg is stale beyond the configured source/market threshold.

### 7.5 Executable prices only

For Polymarket, arbitrage calculations use executable ask/bid/order-book depth appropriate to the intended trade side, not UI midpoint or implied probability alone.

### 7.6 Read-only source discipline

Source probes must observe data that the owner is authorized to access. A blocked source is recorded as blocked; access controls are not bypassed.

## 8. Initial supported universe

The initial whitelist is intentionally narrow:

- sports selected during Phase 1/2 based on overlap evidence;
- prematch first; live added only after source latency/reconnect behavior is characterized;
- binary market families with no unmodeled third settlement state;
- exact line and period equivalence required.

Candidate examples:

- two-way match winner where draw is impossible by rule and void behavior is compatible;
- half-point totals such as O/U 2.5;
- half-point handicaps where settlement semantics match;
- BTTS YES/NO;
- narrowly defined binary propositions.

Integer-line markets with possible pushes are excluded until a refund-aware model is explicitly designed and accepted.

## 9. Success criteria for research phase

The research phase succeeds if the scanner can run for a representative observation window and produce a reproducible dataset containing:

- source coverage and uptime;
- number of discovered events/markets;
- matching funnel counts;
- accepted/rejected/unverified reasons;
- quote update rates and source age distributions;
- theoretical vs validated opportunity counts;
- opportunity lifetime distributions;
- executable size estimates;
- data-quality incidents;
- enough evidence to decide whether Phase 6 expansion is justified.

A result showing that usable arbitrage is rare or absent is still a successful research outcome if the measurement is credible.

## 10. Change control

Any change to the following requires an architect decision recorded in `docs/DECISION_LOG.md`:

- addition of automated wagering;
- new canonical market family;
- new source/provider;
- change to arbitrage formulas or fee model;
- change to settlement compatibility policy;
- migration from SQLite to another primary store;
- change to the branch/review model;
- persistent use of authenticated sportsbook data;
- any access method with material terms/compliance implications.
