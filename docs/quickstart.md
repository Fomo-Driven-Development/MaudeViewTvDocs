# Quick Start

## Prerequisites

- **Go 1.24+** — [install](https://go.dev/dl/)
- **Chromium or Google Chrome** — the browser that tv_controller drives
- **just** — task runner — `cargo install just` or [other methods](https://github.com/casey/just#installation)

For the WatchlistManagerAgent (optional):

- **Python 3.11+**
- **uv** — Python package manager — `curl -LsSf https://astral.sh/uv/install.sh | sh`

## Option A: Use the Core Directly

Install the tv_controller binary and start making API calls.

### 1. Install

```bash
go install github.com/dgnsrekt/MaudeViewTVCore/cmd/tv_controller@latest
```

This places the `tv_controller` binary in your `$GOPATH/bin`.

### 2. Configure

Create a `.env` file in your working directory:

```bash
CHROMIUM_CDP_ADDRESS=127.0.0.1
CHROMIUM_CDP_PORT=9220
CHROMIUM_START_URL=https://www.tradingview.com/
CHROMIUM_PROFILE_DIR=./chromium-profile
CONTROLLER_BIND_ADDR=127.0.0.1:8188
CONTROLLER_LAUNCH_BROWSER=true
CONTROLLER_TAB_URL_FILTER=tradingview.com
CONTROLLER_EVAL_TIMEOUT_MS=5000
CONTROLLER_LOG_LEVEL=info
CONTROLLER_LOG_FILE=logs/tv_controller.log
```

### 3. Run

```bash
CONTROLLER_LAUNCH_BROWSER=true tv_controller
```

This launches Chromium with CDP enabled and starts the API server.

### 4. Log into TradingView

The browser opens to tradingview.com. **Log into your TradingView account.** The controller needs an authenticated session for most operations.

### 5. Make Your First API Call

```bash
# Health check
curl http://127.0.0.1:8188/health

# Get current symbol
curl http://127.0.0.1:8188/api/v1/charts/active

# Change symbol to AAPL
curl -X PUT http://127.0.0.1:8188/api/v1/chart/active/symbol \
  -H "Content-Type: application/json" \
  -d '{"symbol": "AAPL"}'

# List watchlists
curl http://127.0.0.1:8188/api/v1/watchlists
```

### 6. Browse All Endpoints

Open [http://127.0.0.1:8188/docs](http://127.0.0.1:8188/docs) for the interactive API reference, or see the [API Reference](api-reference.md) page.

## Option B: Use with an Agent

The [MaudeViewWatchlistManagerAgent](https://github.com/Fomo-Driven-Development/MaudeViewWatchlistManagerAgent) wraps the REST API into MCP tools for use with Claude.

### 1. Clone

```bash
git clone https://github.com/Fomo-Driven-Development/MaudeViewWatchlistManagerAgent.git
cd MaudeViewWatchlistManagerAgent
```

### 2. Configure

```bash
cp .env.example .env
```

Edit `.env` and add your Anthropic API key (needed for the Claude agent):

```bash
ANTHROPIC_API_KEY=sk-ant-...
```

### 3. Setup

```bash
just setup
```

This builds the Go MCP binary, installs Python dependencies, and installs the tv_controller binary.

### 4. Start the Controller

```bash
just run-tv-controller-with-browser
```

Log into TradingView in the browser that opens.

### 5. Start the Agent (New Terminal)

```bash
just agent
```

This launches an interactive CLI where you can ask Claude to manage your watchlists and charts using the MCP tools.

## Stopping

Press `Ctrl+C` in the controller terminal. The API server shuts down and the browser is terminated automatically.

If you need to kill orphaned processes:

```bash
lsof -ti:8188 | xargs kill -9   # controller
lsof -ti:9220 | xargs kill -9   # browser CDP port
```
