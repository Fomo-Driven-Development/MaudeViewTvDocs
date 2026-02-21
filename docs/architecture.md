# Architecture

## System Overview

```
┌────────────────┐     CDP      ┌────────────────┐   REST API   ┌────────────────┐
│    Chromium    │◄────────────►│ tv_controller  │◄────────────►│  Your Agent    │
│ (TradingView)  │  WebSocket   │  (Go binary)   │  HTTP/JSON   │ (MCP/CLI/...)  │
└────────────────┘              └────────────────┘              └────────────────┘
                                       │
                                       │ JS eval in
                                       │ browser tabs
                                       ▼
                                TradingView's internal
                                JavaScript APIs
```

**Chromium** runs TradingView in a real browser session with your account logged in. The **tv_controller** connects via Chrome DevTools Protocol, evaluates JavaScript in the page context, and exposes the results as a REST API. **Your agent** makes HTTP calls to the controller to automate chart operations.

## Why This Design

### Core as Infrastructure

The tv_controller provides 184 endpoints covering nearly every TradingView operation. It handles the hard parts: CDP connection management, tab discovery, JavaScript evaluation with timeouts, mutex-based chart locking, and error handling.

Your agent doesn't need to know about CDP, WebSockets, or TradingView's internal JavaScript. It just makes HTTP requests.

### Agents as Thin Clients

A single MCP server with 184 tools would be bloated and unfocused. Instead, you build agents tailored to your workflow:

- A **watchlist manager** that uses 15 watchlist endpoints
- A **chart setup agent** that combines symbol + resolution + timeframe + indicators
- A **replay controller** that wraps the 14 replay endpoints
- A **composite tool** that makes multiple API calls in sequence

Each agent picks the endpoints it needs and wraps them as MCP tools, CLI commands, or whatever interface makes sense.

### The WatchlistManagerAgent Pattern

The [MaudeViewWatchlistManagerAgent](https://github.com/Fomo-Driven-Development/MaudeViewWatchlistManagerAgent) demonstrates this pattern. It's a Go MCP server that:

1. Reads `CONTROLLER_BIND_ADDR` from `.env`
2. Wraps selected tv_controller endpoints as MCP tools
3. Each tool is a thin HTTP call — typically 10 lines of Go
4. Adds a Python A2A layer for interactive Claude agent usage

See [Building Agents](building-agents.md) for the full pattern.

## Internal Packages

The tv_controller is built from these internal packages:

| Package | Role |
|---|---|
| `cdpcontrol` | CDP client — connects to browser, evaluates JS in chart tabs, per-chart mutex locking |
| `controller` | Service layer — input validation, error codes, wraps cdpcontrol |
| `api` | Huma REST server — defines HTTP operations, routes, request/response types |
| `config` | Environment variable loading with defaults |
| `browser` | Optional browser launcher — detects and starts Chromium with CDP flags |

## Data Flow

**Read operations** (GET): Agent sends HTTP request to tv_controller, which evaluates JavaScript in the browser tab and returns the result as JSON.

**Write operations** (PUT/POST): Agent sends JSON payload, tv_controller evaluates JavaScript that calls TradingView's internal APIs, and returns the new state.

**CDP keyboard shortcuts**: Some operations (like "reset chart view") use CDP keyboard events instead of JavaScript evaluation, because the corresponding JS APIs have been removed from TradingView.

## The Researcher (Passive Capture)

MaudeViewTVCore also includes a **researcher** binary — a passive traffic capture daemon that records HTTP requests, WebSocket frames, and static resources from TradingView tabs. This is useful for reverse engineering TradingView's internal APIs but is separate from the controller workflow.
