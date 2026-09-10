# Project State — Recovery Cursor

**Updated:** 2026-09-10  
**Repository:** `MaksimUnimax/Test_Poly`  
**Default branch:** `main`

## 1. Project state

```text
PROJECT = Test_Poly
MODE = READ_ONLY_RESEARCH / PAPER_ARBITRAGE
TARGET_OS = Windows
PRIMARY_WORKSPACE = D:\Test_Poly
ARCHITECT = ChatGPT project architect
IMPLEMENTATION_AGENT = local Codex
OWNER = repository / workstation owner
```

## 2. Roadmap cursor

```text
PHASE_0_AUTHORITY_AND_DOCUMENTATION = COMPLETE
PHASE_1_WINDOWS_BOOTSTRAP_AND_SOURCE_PROBES = NOT_STARTED
PHASE_2_DOMAIN_AND_PERSISTENCE = NOT_STARTED
PHASE_3_POLYMARKET_CONNECTOR = NOT_STARTED
PHASE_4_FONBET_COLLECTOR = NOT_STARTED
PHASE_5_MATCHING_AND_WATCHLIST = NOT_STARTED
PHASE_6_ARBITRAGE_AND_EVIDENCE = NOT_STARTED
PHASE_7_TELEGRAM_AND_DIAGNOSTICS = NOT_STARTED
PHASE_8_OBSERVATION_CAMPAIGN = NOT_STARTED
PHASE_9_EXPANSION_DECISION = NOT_STARTED
```

## 3. Current implementation task

```text
CURRENT_TASK = NONE
CURRENT_CODEX_BRANCH = NONE
CURRENT_HANDOFF = NONE
CURRENT_ARCHITECT_REVIEW = NONE
```

No backend implementation has been authorized or started by the architect at this cursor.

## 4. Accepted architecture decisions

Current controlling decisions are `D-001` through `D-014` in `docs/DECISION_LOG.md`.

Most important invariants:

- paper/read-only first; no automated bets/orders;
- Windows-first async Python modular monolith;
- SQLite MVP;
- discovery plane separate from hot watch;
- official Polymarket public market data is primary Polymarket feed;
- Fonbet is `PROBE_ONLY` until Phase 1 evidence determines acquisition method;
- settlement compatibility is a hard match gate;
- unmodeled push/refund markets excluded;
- Polymarket executable ask/depth drives buy-side paper calculation;
- fees/costs are versioned;
- normalized quote history is change-only;
- Codex works through architect-issued task branches/handoffs/review;
- repository-specific deploy key preferred for local Codex;
- `main` is the architect-reviewed integration branch.

## 5. Source status

### Polymarket

```text
ARCHITECTURE_STATUS = APPROVED_PUBLIC_DATA_PATH
IMPLEMENTATION_STATUS = NOT_STARTED
LIVE_PROBE_STATUS = NOT_RUN_IN_THIS_REPOSITORY
```

Planned Phase 1 bounded probe verifies discovery + CLOB book + market WebSocket on the owner’s workstation.

### Fonbet

```text
ARCHITECTURE_STATUS = PROBE_ONLY
PERSISTENT_COLLECTOR_METHOD = UNDECIDED
IMPLEMENTATION_STATUS = NOT_STARTED
LIVE_PROBE_STATUS = NOT_RUN_IN_THIS_REPOSITORY
```

Expected decision after Phase 1:

```text
D-015_FONBET_ACQUISITION_METHOD = PENDING_PROBE_EVIDENCE
```

No one should invent this decision before the browser/network probe.

## 6. External provider status

```text
THIRD_PARTY_ODDS_PROVIDER = NONE_APPROVED
```

A third-party provider may later be evaluated as control/benchmark/fallback only after an architect decision.

## 7. Security cursor

```text
AUTOMATED_WAGERING = FORBIDDEN_CURRENT_ROADMAP
VPN_GEOBLOCK_BYPASS = OUT_OF_SCOPE
ANTI_BOT_BYPASS = OUT_OF_SCOPE
CAPTCHA_BYPASS = OUT_OF_SCOPE
POLYMARKET_PRIVATE_KEY = NOT_REQUIRED_AND_MUST_NOT_BE_ADDED
SPORTSBOOK_ACCOUNT_AUTOMATION = NOT_AUTHORIZED
```

Runtime `.env`, data, logs, DBs and browser profiles are gitignored.

## 8. Local Codex onboarding required before Task 001

Owner/local Codex should establish:

1. repository clone at `D:\Test_Poly`;
2. read/write Git access using the repository-specific SSH deploy key described in `docs/10_WINDOWS_OPERATIONS.md`;
3. successful `git fetch` / `git ls-remote origin`;
4. clean local `main` matching remote;
5. Codex reads `README.md`, `AGENTS.md` and all authority docs.

This is infrastructure onboarding, not permission to begin arbitrary code.

## 9. Next architect action

After local Git access is confirmed:

```text
CREATE TASK_001 = PHASE_1_TOOLING_BOOTSTRAP
```

Recommended Task 001 scope:

- Python project skeleton;
- reproducible Windows setup;
- dependency/config baseline;
- test/lint/type baseline;
- runtime directories/config examples;
- no live provider calls yet unless explicitly included by architect.

Then separate bounded source-probe tasks should follow so each probe has clear evidence and review.

## 10. Next Codex action

```text
NONE UNTIL TASK_001_IS_ISSUED
```

Codex may perform local Git/deploy-key onboarding with the owner, but must not start backend/source implementation merely from the roadmap.

## 11. Open questions

These are intentionally unresolved until evidence exists:

- exact Fonbet transport/acquisition method;
- exact source update latency/staleness thresholds;
- which sports/market families have enough real overlap to prioritize first;
- whether sportsbook max stake can be observed read-only;
- whether a third-party comparison provider is useful;
- whether live monitoring is viable after prematch research;
- whether SQLite needs replacement after measured load.

## 12. Phase 0 closeout

```text
PHASE_0_REMOTE_READBACK = PASS
AUTHORITY_FILES_00_THROUGH_12 = PRESENT_ON_MAIN
COORDINATION_TEMPLATES = PRESENT_ON_MAIN
ROOT_AGENTS_README_GITIGNORE = PRESENT_ON_MAIN
```

Phase 0 is closed. The next engineering action is local Codex/Git onboarding followed by an architect-issued `TASK_001`; no backend code should be started before that task exists.
