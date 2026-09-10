# 05 — Matching and Arbitrage Model

## 1. Core rule

The scanner is allowed to miss opportunities. It is not allowed to manufacture them by matching different propositions.

Matching is therefore a gated pipeline, not a fuzzy-text lookup.

## 2. Matching pipeline

```text
source events
  -> normalization
  -> candidate generation
  -> event hard gates
  -> event confidence/reasons
  -> source markets within accepted event
  -> market-family normalization
  -> rule-signature comparison
  -> outcome mapping
  -> MATCHED / UNVERIFIED / REJECTED
  -> active watchlist
```

## 3. Event candidate generation

Candidate generation may use:

- normalized participant names/aliases;
- competition name/alias;
- start-time proximity;
- sport;
- source identifiers from an approved crosswalk;
- token similarity only as a supporting signal.

Candidate generation is intentionally permissive; acceptance is not.

## 4. Event hard gates

Reject or leave unverified when appropriate if:

- sport differs;
- participant sets cannot be reconciled;
- competition contexts materially conflict;
- start times are outside configured tolerance without a known source-time explanation;
- event scope differs (main match vs reserve/youth/women category confusion, different map/set, etc.);
- one event is live and the other represents a different scheduled event instance.

Participant order may differ between sources; the decision must persist the mapping.

## 5. Market-family normalization

Source labels are mapped into approved canonical families. Initial families:

### 5.1 `MATCH_WINNER_2WAY`

Allowed only where the relevant rules do not introduce an unmodeled draw/third state and settlement policies are compatible.

### 5.2 `TOTAL_2WAY`

Requires the same statistic, exact line and period. Half-point lines are preferred initially because no ordinary equality/push state exists.

Integer lines are excluded until refund/push semantics are implemented and accepted.

### 5.3 `HANDICAP_2WAY`

Requires exact signed line, participant orientation, statistic and period. Half lines preferred initially.

### 5.4 `BTTS`

Requires equivalent “both teams to score” definition and match-period rules.

### 5.5 `BINARY_PROP`

Only approved proposition keys with explicit rule mappings. Free-form proposition matching cannot directly enter the hot watchlist.

## 6. Settlement-compatibility gate

For every family, define `required_rule_fields`.

Comparison returns:

- `COMPATIBLE` — all required fields known and compatible;
- `INCOMPATIBLE` — one or more required fields conflict;
- `UNKNOWN` — one or more required fields are not known with enough confidence.

Mapping:

```text
COMPATIBLE   -> candidate may become MATCHED
INCOMPATIBLE -> REJECTED
UNKNOWN      -> UNVERIFIED
```

No confidence score can transform `UNKNOWN` or `INCOMPATIBLE` into `MATCHED`.

## 7. Examples of false matches to reject

- football “winner in regulation” vs “winner including extra time”;
- tennis winner where one side voids on retirement and the other settles differently;
- total goals 2.5 full match vs first-half total 2.5;
- CS2 match winner vs map 1 winner;
- handicap `Team A -1.5` matched against `Team A +1.5` due to orientation error;
- integer total with push/refund on one source vs binary settlement on another;
- “fighter wins by KO/TKO” vs “fight ends by KO/TKO”;
- same team names in different competitions/start instances.

These negative cases must exist in automated tests.

## 8. Outcome complement mapping

A two-leg cross-platform arbitrage uses complementary outcomes of the **same proposition**.

Example:

```text
Sportsbook: Outcome A wins @ decimal odds o_A
Polymarket: complementary Outcome B token, bought at ask p_B
```

The reverse direction must also be evaluated when data exists:

```text
Sportsbook: Outcome B @ o_B
Polymarket: Outcome A token ask p_A
```

Never compare same-direction outcomes as though they hedge each other.

## 9. Basic no-fee two-leg condition

Assume a target common gross payout `R` in source currency units after normalizing units for paper analysis.

For sportsbook outcome A at decimal odds `o`:

```text
sportsbook stake S = R / o
```

For complementary Polymarket outcome B, one token share pays 1 unit if B resolves true. To target payout `R`:

```text
Polymarket shares Q = R
```

At one executable ask price `p`:

```text
Polymarket cost P = R * p
```

Total capital outlay before modeled fees/costs:

```text
C = R/o + R*p
  = R * (1/o + p)
```

No-fee arbitrage exists only when:

```text
cost_ratio = 1/o + p < 1
```

Store separate metrics so “edge” is never ambiguous:

```text
payout_margin = 1 - cost_ratio
capital_roi = (R - C) / C = 1/cost_ratio - 1
```

Display/reporting must name which metric is shown.

## 10. Depth-aware Polymarket cost

A displayed best ask is not sufficient for a target size.

For desired `Q` shares, consume asks in ascending price order:

```text
for level in asks ascending:
    take = min(level.size, remaining)
    cost += take * level.price
    remaining -= take
```

If depth cannot fill `Q`, that target size is not executable in the paper model.

The engine should solve for the largest target payout/notional that passes minimum ROI and available Polymarket depth, subject to sportsbook size knowledge.

## 11. Sportsbook size

Sportsbook displayed odds do not necessarily reveal the maximum stake available to the owner.

Therefore distinguish:

- `BOOK_SIZE_KNOWN` — an approved read-only observation provides a usable maximum/limit;
- `BOOK_SIZE_UNKNOWN` — do not invent a maximum.

When unknown, report:

```text
validated Polymarket depth = X
sportsbook executable size = UNVERIFIED
```

Alert policy decides whether size-unverified opportunities may be sent. Research storage retains them with the flag.

## 12. Fees and costs

Do not hard-code the assumption that Polymarket is fee-free.

The cost engine receives a versioned cost model. Depending on current market rules, it may calculate:

- trading/taker fee;
- fee nonlinearities by price if applicable;
- source-specific transaction cost assumptions;
- configurable slippage reserve;
- currency conversion cost only if a future model requires it.

The exact formula comes from the current official source documentation and must have unit tests using known examples.

Evaluation stages:

```text
raw book + odds
 -> no-fee theoretical condition
 -> depth simulation
 -> fee/cost simulation
 -> staleness/rule gates
 -> validated paper result
```

## 13. Freshness gate

At evaluation time `t`, determine leg ages using the best trustworthy timestamp available.

A market cannot be validated if:

```text
age_polymarket > stale_limit_polymarket(mode)
OR
age_sportsbook > stale_limit_sportsbook(mode)
```

If a source has no trustworthy source timestamp, use receive time and mark source timestamp unknown.

For live markets, stale limits will likely be stricter than prematch, but exact values must come from measured feed behavior.

## 14. Atomicity / leg risk

The MVP does not place trades, so it cannot claim atomic executability. Every record is paper evidence of simultaneous observed quotes within measured timestamp limits.

Reports/alerts must use language such as `paper opportunity` and must not imply guaranteed fill.

If execution is ever designed, leg risk becomes a separate architecture phase.

## 15. Opportunity open condition

A continuous opportunity episode opens when an evaluation for a specific `market_match_id + direction_key` first satisfies the configured theoretical threshold.

It becomes `VALIDATED_PAPER` only when all required gates pass:

- match is `MATCHED`;
- both legs open/not suspended;
- freshness passes;
- exact line/period rules remain unchanged;
- depth passes minimum requirement;
- post-cost ROI passes threshold;
- required source health is acceptable.

## 16. Opportunity close condition

Expire/reject current episode when any of these becomes true:

- price inequality no longer passes;
- either leg becomes stale;
- market suspends/closes;
- line changes;
- match becomes invalid/unverified;
- required depth disappears;
- source enters a state that invalidates freshness;
- event starts/changes mode under a policy that requires rematching.

Reappearance after expiry creates a new episode or increments an explicitly designed generation counter; choose one deterministic policy and test it.

## 17. Deduplication

An alert fingerprint should include at least:

```text
market_match_id
direction_key
opportunity_episode_id
policy_version
```

Repeated ticks inside one unchanged episode do not send repeated messages unless a material-change policy is configured, e.g. ROI improves by X basis points or size crosses a threshold.

## 18. Matching confidence

A numeric confidence score may rank candidates for analysis but must be decomposable into reasons/features. Do not use an opaque ML model in the MVP as the final acceptance authority.

Initial matching should be deterministic/rule-based with alias dictionaries and explicit tolerances. Fuzzy text assists candidate generation only.

## 19. Manual overrides

A manual match override, if later implemented, must record:

- actor/operator;
- time;
- source event/market IDs;
- previous decision;
- new decision;
- reason;
- rule signature reviewed;
- expiration/revalidation condition.

Manual override does not mean “ignore settlement mismatch”. Architect approval is required for override semantics.

## 20. Research funnel metrics

Persist/export counts at each step:

```text
source events
 -> event candidates
 -> event matches
 -> source markets in matched events
 -> market candidates
 -> MATCHED
 -> UNVERIFIED
 -> REJECTED
 -> active watchlist
 -> theoretical opportunities
 -> cost/depth/freshness validated
 -> alerted
```

This funnel is essential for determining whether failures come from lack of market overlap, poor acquisition, matching uncertainty or lack of price edge.
