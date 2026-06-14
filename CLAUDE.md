# CLAUDE.md — kite-mcp-instruments

`github.com/algo2go/kite-mcp-instruments` — loads + caches the Zerodha Kite **instruments master** (the full tradable-instrument dump from `api.kite.trade/instruments.json`). Part of the algo2go engine for kite-mcp-server (see `../CLAUDE.md`).

## ⚠ Restart / cold-start critical finding (audit 2026-06-14)

**The initial instruments load is SYNCHRONOUS and FATAL, and it gates the consuming server's port-bind.**

- `instruments.New` (production path — `SkipFetch` is only set in tests) calls `UpdateInstruments()` → `loadFromURL()`, which HTTP-GETs `api.kite.trade/instruments.json` (~6–10 MB JSONL, parsed line-by-line) **synchronously during construction** and **returns an error if it fails** (`manager.go:137-142`).
- The consumer builds this inside `initializeServices` **before** binding `:8080` (`../kite-mcp-kc/manager_init.go:30-49` → `instruments.New`; `../kite-mcp-bootstrap` `RunServer` order), so the first inbound request on a cold machine absorbs VM-start + this fetch+parse.
- Retry/timeout: `RetryAttempts=3`, `RetryDelay=3s`, HTTP client `Timeout=30s` (`manager.go:25-26,353`) → **worst case ≈ 96s, after which `New` returns err → the server process exits 1 (crash-loop)** until Kite recovers.

**Impact:** consumer cold start ≈ **12s** typical (measured against the hosted kite-mcp-server). On an ephemeral / scale-to-zero host every wake re-fetches (extra Kite-API load + repeated latency), and a Kite-API hiccup at wake crash-loops boot.

**Recommended fix (DEFERRED — the highest-value follow-up from the audit):** make the initial load **async / non-fatal** — start with an empty map and let the existing background refresh populate it, instead of returning an error on first-fetch failure. This removes the crash-loop and shrinks cold start. Lower-effort interim: reduce `RetryAttempts`/`RetryDelay` for hosted builds so worst-case boot can't reach ~96s.

Full context: `../../kite-mcp-server/.research/2026-06-14-scale-to-zero-safety-audit.md` (Lens 6).
