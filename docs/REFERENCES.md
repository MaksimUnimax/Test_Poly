# References and External Authority

This file records external documentation used by the architecture. External behavior can change; implementation tasks must recheck current docs when the referenced behavior materially affects code.

**Bootstrap review date:** 2026-09-10

## Polymarket

### Public market data overview

https://docs.polymarket.com/market-data/overview

Architecture use:

- official public market-data APIs;
- no API key/authentication/wallet required for the documented public market-data path;
- Gamma events/markets/sports metadata;
- CLOB price/order-book endpoints.

### Market WebSocket

https://docs.polymarket.com/api-reference/wss/market

Architecture use:

- public market channel;
- token/asset subscriptions;
- order-book snapshots;
- price-level changes;
- market lifecycle/heartbeat behavior.

### API rate limits

https://docs.polymarket.com/api-reference/rate-limits

Architecture use:

- source-specific bounded request/concurrency policy;
- avoid unnecessary REST polling when WebSocket is sufficient.

### Trading fees

https://docs.polymarket.com/trading/fees

Architecture use:

- fees may be enabled per market/category;
- fee calculation must be versioned/current rather than assumed zero;
- paper arbitrage needs post-cost evaluation.

### Fee-rate endpoint

https://docs.polymarket.com/api-reference/market-data/get-fee-rate

Architecture use:

- source of per-token/current fee-rate information when implementing cost model.

### Trading overview — execution boundary reference

https://docs.polymarket.com/trading/overview

Architecture use:

- confirms trading is a separate authenticated/order-signing concern;
- MVP intentionally excludes wallet/order credentials and execution code.

## Browser/network observation

### Playwright Python — Network

https://playwright.dev/python/docs/network

Architecture use:

- HTTP request/response observation;
- XHR/fetch observation;
- WebSocket creation/frame inspection;
- bounded headful browser probe for sportsbook acquisition research.

### Chrome DevTools Protocol — Network domain

https://chromedevtools.github.io/devtools-protocol/tot/Network/

Architecture use:

- optional lower-level diagnostic supplement for browser network/WebSocket inspection when required.

## Fonbet

### International Fonbet rules

https://fonbet.com/rules

Architecture use:

- terms/access review reference for the international `fonbet.com` operator only;
- current rules contain restrictions relevant to automated tools in the context of making bets and automated request submission;
- MVP keeps read-only observation and any future execution strictly separate.

Important: do not treat this URL as authority for a different Fonbet legal entity/domain without checking that operator’s actual terms.

## GitHub

### Deploy keys

https://docs.github.com/en/rest/deploy-keys/deploy-keys

Architecture use:

- repository-scoped SSH key model for local Codex Git access;
- public key attached to one repository; private key remains on local machine.

## Telegram

### Telegram bots developer documentation

https://core.telegram.org/bots

Architecture use:

- Bot API integration for notifications;
- bot token is a secret and must remain outside Git/logs.

### Bot API reference

https://core.telegram.org/bots/api

Architecture use:

- notification transport implementation and API behavior.

## SQLite

### SQLite documentation

https://www.sqlite.org/docs.html

### Write-Ahead Logging

https://www.sqlite.org/wal.html

Architecture use:

- local MVP persistence;
- WAL/backup/checkpoint semantics must follow SQLite documentation rather than ad-hoc file copying assumptions.

## Python

### `decimal` module

https://docs.python.org/3/library/decimal.html

Architecture use:

- decimal odds, prices, money and fee arithmetic should not rely on binary floating-point semantics.

### `asyncio`

https://docs.python.org/3/library/asyncio.html

Architecture use:

- async source I/O and bounded event-driven hot path.

## Source review rule

When a current external fact affects an implementation decision, the implementation task must record the exact document/endpoint/version/date observed. In particular recheck before implementation:

- Polymarket fee behavior;
- Polymarket WebSocket message schema;
- API rate limits;
- Fonbet/source acquisition behavior and applicable terms;
- Telegram API constraints if alert behavior depends on them.

Do not copy third-party marketing claims into architecture as execution truth without independent measurement.
