<p align="center">
  <img src="assets/banner.jpg" alt="MaudeView" width="600">
</p>

# MaudeView

**Browser automation for TradingView via Chrome DevTools Protocol.**

MaudeView gives you programmatic control over TradingView charts through a REST API. Change symbols, manage watchlists, draw on charts, run Pine scripts, control replay mode, manage alerts — all through HTTP calls to a local server that drives a real browser.

## The Idea

TradingView is powerful but its automation options are limited. MaudeView bridges that gap by connecting to a Chromium browser via CDP and evaluating JavaScript directly in TradingView's pages. The result is 177 REST endpoints covering nearly every chart operation.

But 177 endpoints in a single tool would be unwieldy. Instead of one monolithic agent, MaudeView is designed as a **toolkit**:

- **[MaudeViewTVCore](https://github.com/Fomo-Driven-Development/MaudeViewTVCore)** is the engine — install it, run it, and you have a REST API for TradingView
- **You build agents** that call the endpoints you need, packaged as MCP tools, CLI scripts, or whatever fits your workflow
- **[MaudeViewWatchlistManagerAgent](https://github.com/Fomo-Driven-Development/MaudeViewWatchlistManagerAgent)** is a reference implementation — a working agent you can study, fork, or use directly

This keeps each agent focused and your tooling lean.

## What You Can Do

- **Chart control** — set symbols, resolutions, chart types, timeframes, navigate to dates
- **Watchlist management** — create, rename, delete watchlists, manage symbols, colored lists
- **Drawing tools** — place lines, shapes, text, tweets, icons on charts
- **Studies & indicators** — add, remove, configure technical indicators
- **Layout management** — save, clone, rename, switch between multi-chart layouts
- **Pine Editor** — read/write Pine scripts, save, compile, add to chart
- **Replay mode** — activate, step through bars, autoplay at custom speeds
- **Alerts** — create, modify, delete, start/stop alerts
- **Snapshots** — capture chart screenshots programmatically
- **Strategy testing** — read reports, navigate to dates, configure inputs

## Next Steps

- [Architecture](architecture.md) — how the pieces fit together
- [Quick Start](quickstart.md) — get running in 5 minutes
- [API Reference](api-reference.md) — browse all 177 endpoints
- [Building Agents](building-agents.md) — create your own agent
