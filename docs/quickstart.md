# Quick Start

## Prerequisites

- **Go 1.24+** — [install](https://go.dev/dl/)
- **Chromium or Google Chrome** — the browser that tv_controller drives
- **just** — task runner — `cargo install just` or [other methods](https://github.com/casey/just#installation)
- **Claude Code** — install with `npm install -g @anthropic-ai/claude-code` (requires a [Claude account](https://claude.ai/) with a Max plan or API access)

For the WatchlistManagerAgent's Python A2A layer (optional):

- **Python 3.11+**
- **uv** — Python package manager — `curl -LsSf https://astral.sh/uv/install.sh | sh`
- **Anthropic API key** — get one at [console.anthropic.com](https://console.anthropic.com/)

## Option A: Use with an Agent

The fastest way to get started. The [MaudeViewWatchlistManagerAgent](https://github.com/Fomo-Driven-Development/MaudeViewWatchlistManagerAgent) wraps the REST API into MCP tools that Claude can use to control TradingView.

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

This builds the Go MCP binary, installs Python dependencies, and installs the tv_controller binary from MaudeViewTVCore.

### 4. Start the Controller

```bash
just run-tv-controller-with-browser
```

This launches Chromium and starts the REST API server. **Log into TradingView** in the browser that opens.

### 5. Start the Agent (New Terminal)

=== "Claude Code (recommended)"

    If you have Claude Code installed, you can use the MCP tools directly in your terminal:

    ```bash
    claude
    ```

    Claude Code will discover the MCP server via `.mcp.json` and have access to all 50 watchlist and chart tools.

=== "Interactive CLI"

    ```bash
    just agent
    ```

    This launches a standalone CLI using the Claude Agent SDK where you can ask Claude to manage your watchlists and charts.

## Option B: Use the Core Directly

Install just the tv_controller binary and make API calls yourself — with curl, scripts, or your own tools.

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

This launches Chromium with CDP enabled and starts the API server. **Log into TradingView** in the browser that opens.

### 4. Make Your First API Call

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

### 5. Browse All Endpoints

Open [http://127.0.0.1:8188/docs](http://127.0.0.1:8188/docs) for the interactive API reference, or see the [API Reference](api-reference.md) page.

## Stopping

Press `Ctrl+C` in the controller terminal. The API server shuts down and the browser is terminated automatically.

If you need to kill orphaned processes:

```bash
lsof -ti:8188 | xargs kill -9   # controller
lsof -ti:9220 | xargs kill -9   # browser CDP port
```
