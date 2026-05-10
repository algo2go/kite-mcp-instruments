# kite-mcp-instruments

[![Go Reference](https://pkg.go.dev/badge/github.com/algo2go/kite-mcp-instruments.svg)](https://pkg.go.dev/github.com/algo2go/kite-mcp-instruments)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Kite instruments fetcher + cache for the algo2go ecosystem. Provides
NSE/BSE symbol-to-token resolution, search by symbol/exchange/segment,
TTL-based cache refresh aligned to market hours, and `Manager`
lifecycle management (Start/Stop/Refresh).

Used by [`Sundeepg98/kite-mcp-server`](https://github.com/Sundeepg98/kite-mcp-server)
across the broker-services + MCP tool layer for symbol resolution
(buy/sell/search tools), options-chain construction, market-hours
gating, and Telegram trading commands.

## Why a separate module?

Kite instruments is a foundational primitive any algo2go consumer
that places orders or queries market data needs independent of
`kite-mcp-server`. Hosting as a module:

- Centralizes the Kite instruments source-of-truth across consumers
- Lets the cache + refresh schedule version independently
- Pairs cleanly with `algo2go/kite-mcp-isttz` (market-hours alignment)
  for the broker-data primitives stack

## Stability promise

**v0.x — unstable.** Type signatures may evolve. Pin `v0.1.0`
deliberately.

## Install

```bash
go get github.com/algo2go/kite-mcp-instruments@v0.1.0
```

## Public API

- `Manager` — lifecycle-managed instruments fetcher with cache
- `Cache` — symbol-to-token map with TTL-based refresh
- `Instrument` — DTO struct (Token, Symbol, Exchange, Segment, etc.)
- Search helpers: `BySymbol`, `ByExchange`, `BySegment`, `ByToken`
- IST market-hours-aligned refresh (every market-open + EOD)

## Dependencies

- `github.com/algo2go/kite-mcp-isttz` v0.1.0 — IST timezone + market hours
- `github.com/stretchr/testify` — assertions
- `go.uber.org/goleak` — goroutine-leak detection in tests

All algo2go deps are published modules; no upstream `replace`
directives needed.

## Test caveat

Some tests (`TestNew_*InstrumentsManager*`, `TestNewConfigConstructor`,
`TestManager_MoreAccessors`) hit `api.kite.trade` for live instrument
fetch. They fail under WSL2 DNS resolution but pass on Fly.io BOM
region with direct egress. These are pre-existing CI-environment-
specific flakes documented across F1-F7 + 5/5 module dispatches in
the parent repo.

## Reference consumer

[`Sundeepg98/kite-mcp-server`](https://github.com/Sundeepg98/kite-mcp-server)
— consumed across kc/manager_*, kc/options.go, kc/broker_services.go,
kc/ports/instrument.go, kc/ops/scanner.go, kc/ops/payoff.go,
kc/telegram/bot.go, mcp/market_tools.go, mcp/trade/option_tools.go,
mcp/trade/options_greeks_tool.go, mcp/alerts/alert_tools.go.

## License

MIT — see [LICENSE](LICENSE).

## Authors

Original design: [Sundeepg98](https://github.com/Sundeepg98) (Zerodha
Tech). Multi-module promotion (2026-05-10): algo2go contributors.
