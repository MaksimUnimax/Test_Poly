# 03 — Canonical Data Contracts

## 1. Purpose

Source adapters may change; the research core must not. This document defines the canonical entities that separate acquisition from matching, pricing and evidence.

All persistent/domain timestamps are UTC. Use timezone-aware datetimes. Duration measurements within one process should use a monotonic clock in addition to wall time when needed.

## 2. Identifier rules

Every persistent entity has:

- internal UUID or stable application ID;
- source name where applicable;
- original source identifier(s);
- first-seen and last-seen timestamps;
- schema/version provenance when transformation behavior can change.

Never build global identity by concatenating display names alone.

## 3. Enumerations

Minimum enums:

```text
Source = POLYMARKET | FONBET | ...approved future sources
Mode = PREMATCH | LIVE
SourceHealth = DISABLED | STARTING | HEALTHY | DEGRADED | BLOCKED | FAILED | RESYNCING
MatchStatus = CANDIDATE | UNVERIFIED | MATCHED | REJECTED | SUSPENDED | CLOSED
OpportunityStatus = THEORETICAL | REJECTED_STALE | REJECTED_RULES | REJECTED_DEPTH | REJECTED_COSTS | SIZE_UNVERIFIED | VALIDATED_PAPER | EXPIRED
MarketFamily = MATCH_WINNER_2WAY | TOTAL_2WAY | HANDICAP_2WAY | BTTS | BINARY_PROP | ...approved
Period = FULL_EVENT | FIRST_HALF | SECOND_HALF | SET | MAP | ROUND | OTHER
QuoteState = OPEN | SUSPENDED | CLOSED | UNKNOWN
```

Do not add market families ad hoc inside source parsers. New families require a defined rule signature and tests.

## 4. SourceEvent

Represents one source-native event before global matching.

Required fields:

```text
id
source
source_event_id
sport
competition_raw
participants_raw[]
start_time_source
start_time_utc
mode
status_raw
url_or_locator?           # safe non-secret source locator if available
source_updated_at?
received_at_utc
raw_payload_ref?
parser_version
```

No canonical participant equivalence is assumed yet.

## 5. CanonicalEventCandidate

Normalized source event used for cross-source candidate matching.

```text
id
source_event_id_internal
sport
competition_key?
participant_keys[]
participant_display[]
start_time_utc
mode
normalization_version
normalization_notes[]
```

For two-sided sports, participant orientation should be represented explicitly where meaningful (home/away, player1/player2), but the matcher must account for sources that reverse display order.

## 6. SourceMarket

```text
id
source
source_event_id
source_market_id
market_name_raw
outcomes_raw[]
line_raw?
period_raw?
status_raw
rules_text_or_ref?
source_updated_at?
received_at_utc
raw_payload_ref?
parser_version
```

## 7. RuleSignature

Settlement semantics are represented structurally. Required fields vary by market family; unknown must be explicit.

Base fields:

```text
market_family
period
line?                     # Decimal, not binary float
participant_or_team_ref?
outcome_semantics[]
includes_overtime: YES | NO | UNKNOWN | NOT_APPLICABLE
push_possible: YES | NO | UNKNOWN
refund_policy_key?
void_policy_key?
retirement_policy_key?
walkover_policy_key?
extra_time_policy_key?
source_rules_version_or_observed_at?
family_specific: map
```

Examples of family-specific fields:

### MATCH_WINNER_2WAY

```text
winner_scope
retirement_treatment      # important for tennis
walkover_treatment
includes_overtime         # important in some sports
```

### TOTAL_2WAY

```text
statistic                  # goals, games, maps, rounds, points
line
period
includes_overtime
push_possible
```

### HANDICAP_2WAY

```text
statistic
line
participant_side
period
includes_overtime
push_possible
```

### BINARY_PROP

```text
proposition_key
qualifier(s)
period
```

If a required field is `UNKNOWN`, matcher policy decides `UNVERIFIED`; it must not silently default to a convenient value.

## 8. CanonicalMarketCandidate

```text
id
source_market_id_internal
canonical_event_candidate_id
market_family
rule_signature
canonical_outcomes[]
normalization_version
normalization_notes[]
```

## 9. EventMatchDecision

```text
id
left_event_id
right_event_id
status
confidence_score?         # 0..1 informational
hard_gate_results{}
reason_codes[]
participant_mapping{}
start_time_delta_ms
created_at_utc
updated_at_utc
matcher_version
manual_override?          # default false; any override must be audited
```

Suggested reason codes:

```text
SPORT_MISMATCH
PARTICIPANT_MISMATCH
AMBIGUOUS_PARTICIPANT
COMPETITION_CONFLICT
START_TIME_OUT_OF_TOLERANCE
MODE_CONFLICT
EVENT_MATCH_ACCEPTED
```

## 10. MarketMatchDecision

```text
id
event_match_id
left_market_id
right_market_id
status
market_family
rule_compatibility
outcome_mapping{}
reason_codes[]
created_at_utc
updated_at_utc
matcher_version
```

Important reason codes:

```text
FAMILY_MISMATCH
LINE_MISMATCH
PERIOD_MISMATCH
OUTCOME_MAPPING_AMBIGUOUS
OVERTIME_RULE_MISMATCH
PUSH_RULE_MISMATCH
RETIREMENT_RULE_MISMATCH
VOID_RULE_UNVERIFIED
REQUIRED_RULE_UNKNOWN
SETTLEMENT_COMPATIBLE
```

## 11. CanonicalQuote

Represents a current executable/displayed source quote for one outcome/side.

```text
id
source
source_market_id
canonical_market_match_id?
outcome_key
quote_type                # DECIMAL_ODDS | ORDERBOOK_ASK | ORDERBOOK_BID | ...
price_decimal             # Decimal
available_size?           # source-native unit + normalized notional where possible
currency?
quote_state
source_timestamp_utc?
received_at_utc
processed_at_utc
sequence_or_version?
raw_update_ref?
```

For sportsbook odds, `price_decimal` is decimal odds. For Polymarket order-book levels, price is cost per $1 payout token. Domain code must never confuse these representations simply because both are decimals.

## 12. OrderBookSnapshot

```text
id
source_market_id
asset_or_outcome_id
bids[] = {price, size}
asks[] = {price, size}
source_timestamp_utc?
received_at_utc
snapshot_reason           # RESYNC | OPPORTUNITY_EVIDENCE | PERIODIC_DEBUG
```

Normal operation may maintain only required depth in memory; full snapshots are preserved when evidence policy requires them.

## 13. Quote freshness

Derived fields used in evaluation:

```text
source_age_ms = evaluation_wall_time - source_timestamp (when trustworthy)
receive_age_ms = evaluation_wall_time - received_at
processing_delay_ms = processed_at - received_at
```

When source timestamp is absent/untrusted, freshness policy uses receive age and records that source age is unknown.

## 14. CostModelVersion

```text
id
source
market_class?
valid_from_utc
valid_to_utc?
fee_formula_type
fee_parameters{}
slippage_reserve_bps
notes
source_reference?
```

Do not overwrite history when fees change; create a new version.

## 15. OpportunityEvaluation

One evaluation event:

```text
id
market_match_id
evaluated_at_utc
polymarket_leg{}
sportsbook_leg{}
quote_ages{}
cost_model_ids[]
theoretical_cost_ratio
theoretical_edge_fraction
validated_edge_fraction?
max_polymarket_size?
max_sportsbook_size?
validated_executable_notional?
status
reason_codes[]
calculator_version
```

Use `Decimal` for money/probability/odds arithmetic. Define rounding only at display/export boundaries unless source tick rules require it.

## 16. OpportunityLifecycle

Represents one continuous opportunity episode.

```text
opportunity_id
market_match_id
direction_key
first_theoretical_at_utc
first_validated_at_utc?
last_valid_at_utc?
expired_at_utc?
max_theoretical_edge
max_validated_edge?
max_validated_size?
evaluation_count
quote_change_count
current_status
open_reason
close_reason?
```

A direction key distinguishes, for example, “bookmaker A outcome + Polymarket opposite outcome” from the reverse direction.

## 17. AlertRecord

```text
id
opportunity_id
policy_version
trigger_reason
created_at_utc
send_attempted_at_utc?
sent_at_utc?
telegram_message_id?
status
error_class?
payload_hash
```

Do not store Telegram bot token or private chat secrets in this record.

## 18. SourceHealthSample / Incident

```text
source
state
observed_at_utc
last_good_update_utc?
queue_depth?
reconnect_count?
error_class?
message_redacted?
```

Persistent incidents are created for material transitions such as healthy->blocked, disconnect gaps, parser bursts, queue overflow, clock anomaly.

## 19. RawEnvelope metadata

If raw data is retained:

```text
id
source
capture_type
captured_at_utc
content_hash
redaction_version
storage_path
expires_at_utc
content_encoding
```

Raw HTTP headers/cookies/tokens must be removed before persistence unless a specifically approved secure diagnostic requires them, and such artifacts must remain outside Git.

## 20. Schema compatibility rule

Persistent schema changes require migrations and replay/upgrade tests. A field rename or semantic change is not “just refactoring” once data has been recorded.
