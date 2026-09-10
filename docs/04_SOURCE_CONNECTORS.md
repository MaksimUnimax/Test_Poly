# 04 — Source Connectors and Acquisition Strategy

## 1. Principle

External acquisition is replaceable infrastructure. The domain core must not know whether a sportsbook quote came from a public JSON endpoint, a browser-observed XHR response, a WebSocket frame, DOM extraction, or an approved third-party provider.

Every connector must emit the canonical source contracts and explicit provenance.

## 2. Acquisition priority

For a source that does not provide a documented public feed, investigate in this order:

1. documented/public API or feed;
2. documented/public WebSocket/push stream;
3. structured network responses already used by a normally accessed public web page (XHR/fetch/WebSocket), observed through the browser;
4. structured state embedded in the page;
5. DOM extraction as fallback;
6. screenshot/OCR only as a last diagnostic fallback, not a preferred hot feed.

Do not bypass authentication, CAPTCHA, geoblocks, anti-bot controls or access restrictions to move up this list.

## 3. Polymarket connector

### 3.1 Current authority

At project bootstrap, official Polymarket documentation states that market data is available through public REST endpoints without API key, authentication or wallet. Official documentation also exposes a public market WebSocket for real-time order-book, price and lifecycle updates.

References:

- https://docs.polymarket.com/market-data/overview
- https://docs.polymarket.com/api-reference/wss/market
- https://docs.polymarket.com/api-reference/rate-limits

These references must be rechecked if integration behavior changes.

### 3.2 Discovery

Use Gamma/metadata APIs for:

- sports metadata;
- events;
- markets;
- start times;
- descriptions/rules metadata;
- outcome/token identifiers;
- market active/closed state.

The discovery collector must paginate rather than assume one response is complete.

### 3.3 Hot watch

For matched token IDs, prefer the official market WebSocket.

Handle:

- initial book snapshots;
- price-level updates;
- best bid/ask updates when available;
- last-trade events as informational data;
- market lifecycle events;
- heartbeat;
- reconnect/resync.

The arbitrage calculation uses executable order-book sides/depth, not last-trade or midpoint unless a formula explicitly calls for those as diagnostics.

### 3.4 REST fallback / verification

CLOB REST can be used for:

- snapshot/resync;
- book verification on opportunity detection;
- debugging;
- historical/reference price queries where appropriate.

Do not poll REST aggressively when WebSocket data already provides the required state. Respect current documented limits and implement backoff.

### 3.5 Polymarket US vs international data

Do not silently substitute Polymarket US contracts/data for international Polymarket contracts. They are distinct platforms/interfaces. The initial research target is the data universe explicitly configured by the architect.

## 4. Fonbet connector — Phase 1 is a probe, not an assumption

We do not yet freeze a production acquisition method for Fonbet.

The first implementation task for Fonbet must determine how the normally accessed site obtains useful line/odds data and whether that method is stable enough and permitted for our read-only research use.

### 4.1 Probe tool

Use Playwright with Chromium in a normal visible (`headful`) browser session on the owner’s Windows machine.

Playwright officially supports observation of:

- HTTP requests/responses;
- XHR/fetch traffic;
- WebSocket creation;
- WebSocket sent/received frames.

Reference:

- https://playwright.dev/python/docs/network

Chrome DevTools Protocol may be used as a diagnostic supplement when Playwright does not expose enough detail.

### 4.2 Probe outputs

A bounded probe should produce redacted diagnostics such as:

```text
data/raw/fonbet_probe/<run_id>/
  probe_manifest.json
  requests.jsonl
  responses_index.jsonl
  websockets.jsonl
  websocket_frames.jsonl
  page_observations.jsonl
  SUMMARY.md
```

Runtime output lives under gitignored `data/`; only deliberately sanitized fixtures/summaries may later enter `tests/fixtures/`.

### 4.3 Probe questions

The probe must answer:

1. Can the target page be opened normally on the owner’s machine?
2. Is login required for the line data we need?
3. Are prematch events and odds available in structured network data?
4. Are live events and odds available in structured network data?
5. Is there a push/WebSocket channel or repeated polling?
6. What stable source IDs exist for events, markets and selections?
7. How are suspension/closure changes represented?
8. Does the source expose server/source timestamps or sequence numbers?
9. How often do updates occur for a small sample?
10. Is the payload complete enough to reconstruct event + market + side + line + odds?
11. What rate/traffic does normal browser operation generate?
12. Are there access restrictions or terms issues that block use?

### 4.4 Probe stop conditions

Stop and record `BLOCKED` rather than bypass when encountering:

- CAPTCHA/anti-bot gate that prevents normal access;
- mandatory authentication not explicitly authorized for the probe;
- geoblock/access-control barrier;
- a technical requirement to defeat TLS/app protections;
- clear source terms incompatibility with the intended collector mode.

### 4.5 Fonbet terms boundary

The current international `fonbet.com` rules include restrictions on automated tools in the context of making bets and automated request submission. This project does not automate betting. Nevertheless, applicable terms depend on the actual operator/site used by the owner and must be revalidated before converting a diagnostic probe into a persistent collector.

Reference at bootstrap:

- https://fonbet.com/rules

No implementation should generalize this reference to a different legal operator without checking that operator’s terms.

## 5. Direct structured collector after the probe

If the probe establishes a suitable read-only structured stream, the architect may approve a Fonbet collector that:

- runs the browser if the browser session is required;
- subscribes/observes only the relevant sports/markets where practical;
- parses source updates;
- emits source contracts;
- never performs wager actions;
- has deterministic fixture-based tests;
- respects observed update cadence and bounded resource use.

Whether the collector accesses network messages through Playwright, CDP, or a documented endpoint is a post-probe decision recorded in `DECISION_LOG.md`.

## 6. Third-party odds providers

A third-party provider can be useful as:

- a bootstrap source;
- a comparison/control feed;
- a fallback source;
- a latency/coverage benchmark.

But it is not automatically trusted as execution truth.

Before adding one, record:

- provider/source coverage;
- whether Fonbet and Polymarket are actually included;
- update latency;
- timestamp semantics;
- live vs prematch coverage;
- market depth/limits available;
- pricing/licensing/terms;
- rate limits;
- mapping quality;
- whether quotes are direct or transformed.

A third-party provider requires an architect decision before implementation.

## 7. Collector provenance

Every emitted record must identify:

```text
source
acquisition_method
collector_version
parser_version
received_at_utc
source_timestamp_utc if available
raw_payload_ref if retained
```

Suggested acquisition methods:

```text
PUBLIC_REST
PUBLIC_WEBSOCKET
BROWSER_XHR
BROWSER_WEBSOCKET
BROWSER_EMBEDDED_STATE
BROWSER_DOM
THIRD_PARTY_API
REPLAY_FIXTURE
```

## 8. Rate discipline

Each connector owns a configurable rate policy:

- concurrency limit;
- polling interval;
- timeout;
- retryable status/error classes;
- exponential backoff with jitter;
- maximum retry delay;
- circuit-breaker/degraded behavior if appropriate.

No connector may implement rate-limit evasion or proxy rotation for the purpose of defeating source controls.

## 9. Browser profile policy

Browser profiles/cookies may contain private/session data.

Rules:

- store only outside the repository in a gitignored directory;
- never copy profile databases/cookies into fixtures;
- do not log cookie/header values;
- default Phase 1 public-data probe to a clean dedicated browser context unless the owner explicitly authorizes another session;
- if an authenticated session becomes necessary, that is a new decision boundary.

## 10. DOM fallback policy

DOM extraction is accepted only when structured acquisition is unavailable or incomplete.

Selectors should prefer stable semantic attributes over layout-based `nth-child` chains. Parser tests must use captured sanitized HTML fixtures. A site redesign must fail observably rather than silently emitting wrong odds.

## 11. OCR fallback policy

OCR is not an MVP hot-feed architecture. It may be used only to diagnose/confirm information that cannot be obtained structurally, unless the architect later approves a specific OCR collector with an error model and validation gates.

## 12. Source onboarding checklist

A source is not `SUPPORTED` until it has:

- approved acquisition method;
- event/market/quote parser;
- stable source IDs or explicit synthetic-ID policy;
- timestamp semantics documented;
- suspension/closure handling;
- rate/reconnect policy;
- fixture suite;
- normalizer mapping;
- settlement-rule source/reference strategy;
- health checks;
- bounded raw diagnostics;
- source-specific acceptance test.
