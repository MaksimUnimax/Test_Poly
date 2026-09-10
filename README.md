# Test_Poly — Polymarket ↔ Sportsbook Research Scanner

Research-first Windows scanner for finding and measuring price discrepancies between Polymarket sports markets and sportsbook markets, starting with Fonbet.

## Project status

**Status:** architecture/documentation bootstrap.

**Current phase:** Phase 0 — repository authority, local Windows environment, source probes, and read-only data acquisition.

No automated betting is in scope. The first product is a read-only / paper-arbitrage scanner that records evidence about whether executable cross-platform opportunities actually exist, how long they survive, and at what size.

## Core goal

Build a local service that:

1. discovers sports events and markets from Polymarket and one or more sportsbooks;
2. normalizes source-specific data into one canonical model;
3. matches only settlement-compatible markets;
4. watches matched markets with low latency;
5. calculates executable cross-platform arbitrage after fees, freshness and available depth;
6. records opportunity lifetime and supporting snapshots;
7. sends Telegram alerts for opportunities that pass configured safety/quality gates.

## Initial scope

- Platform A: Polymarket public market-data APIs and public market WebSocket.
- Platform B: Fonbet research collector, with acquisition method determined by Phase 1 source probing.
- Operating system: Windows, local workstation.
- Runtime: Python asynchronous backend.
- Storage: SQLite for the MVP, with a migration path to PostgreSQL if needed.
- Notifications: Telegram Bot API.
- Execution: paper only; no wager placement, no automatic order placement, no account automation.

## Binary-market rule

The scanner is designed around markets that can be represented as two complementary outcomes **with compatible settlement rules on both platforms**.

A market is not considered safely binary merely because the UI shows two buttons. Push/refund, overtime, retirement, void and event-cancellation rules are part of the market identity. Integer totals/handicaps or other markets with a possible push/refund state are excluded by default until an explicit settlement model is implemented and approved.

Examples of initially eligible market families, subject to rule-equivalence validation:

- tennis / combat-sport winner markets without a draw state;
- over/under on non-push lines such as 2.5;
- both teams to score YES/NO;
- binary propositions such as “fighter wins by KO/TKO: yes/no” when both sources define the proposition equivalently;
- selected two-way esports markets.

Three-way 1/X/2 markets are out of scope for the MVP.

## Architecture in one picture

```text
                  DISCOVERY PLANE

     Polymarket                         Sportsbook
   Gamma / metadata                    source probe
          |                                |
          +------------+-------------------+
                       |
                 NORMALIZATION
                       |
                EVENT MATCHING
                       |
                MARKET MATCHING
                       |
                ACTIVE WATCHLIST
                       |

                   HOT PATH

 Polymarket CLOB/WSS <-----> Quote State <-----> Sportsbook collector
                                  |
                           Arbitrage Engine
                                  |
                    +-------------+-------------+
                    |                           |
             Evidence Storage              Telegram
```

The discovery plane may be slower. The hot path watches only already matched markets and recalculates only the affected pair when one leg changes.

## Documentation authority

Before coding, read the documents under `docs/` and root `AGENTS.md`.

Order of authority:

1. `docs/00_PROJECT_CHARTER.md`
2. `docs/01_PRODUCT_SPECIFICATION.md`
3. `docs/02_SYSTEM_ARCHITECTURE.md`
4. `docs/03_DATA_CONTRACTS.md`
5. `docs/04_SOURCE_CONNECTORS.md`
6. `docs/05_MATCHING_AND_ARBITRAGE.md`
7. `docs/06_STORAGE_AND_OBSERVABILITY.md`
8. `docs/07_SECURITY_COMPLIANCE_AND_BOUNDARIES.md`
9. `docs/08_TEST_STRATEGY.md`
10. `docs/09_ROADMAP.md`
11. `docs/10_WINDOWS_OPERATIONS.md`
12. `docs/11_CODEX_WORK_PROTOCOL.md`
13. `docs/12_ACCEPTANCE_GATES.md`
14. `docs/PROJECT_STATE.md`
15. `docs/DECISION_LOG.md`
16. `docs/REFERENCES.md`

If implementation conflicts with these documents, implementation is wrong unless the architect has first changed the relevant authority document.

## Roles

- **Architect / technical lead:** ChatGPT in the project conversation. Owns architecture, planning, technical decisions, task decomposition, review and acceptance.
- **Implementation agent:** Codex on the user's Windows machine. Writes code, runs approved probes/tests, reports evidence, commits and pushes. Codex does not independently redefine architecture or project scope.
- **Owner/operator:** repository owner. Controls credentials, local machine access, external accounts and final operational use.

## Repository workflow

- Default branch: `main`.
- No force pushes.
- Codex should normally work on task branches named `codex/<phase>-<short-task>` unless explicitly instructed otherwise.
- Every implementation task must end with tests, a concise handoff, commit SHA and clean working tree.
- Secrets, browser profiles, raw credentials and local data must never be committed.

## Non-goals for the first release

- automatic wager placement;
- bypassing geoblocks, authentication, anti-bot controls or access restrictions;
- guaranteeing profitability;
- treating visually similar markets as equivalent without settlement-rule validation;
- scraping every market continuously when only a small matched watchlist is needed;
- supporting 1/X/2 arbitrage.

## First milestone

The first meaningful milestone is not “Telegram sends a signal”. It is a reproducible dataset proving:

- how many same-event/same-market pairs can be matched;
- what data latency each source has;
- how many theoretical opportunities occur;
- how many survive freshness, fee and settlement checks;
- how much executable volume exists;
- how long each opportunity remains available.

Only after that evidence exists should the project consider broader sportsbook coverage or any execution-oriented feature.
