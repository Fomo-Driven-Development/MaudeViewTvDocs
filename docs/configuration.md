# Configuration

All configuration is via environment variables, loaded from a `.env` file in the working directory.

## CDP Settings

These control how the browser is launched and how the controller connects to it.

| Variable | Default | Description |
|---|---|---|
| `CHROMIUM_CDP_ADDRESS` | `127.0.0.1` | Address the browser listens on for CDP connections. Always use `127.0.0.1` for security. |
| `CHROMIUM_CDP_PORT` | `9220` | Port for Chrome DevTools Protocol. |
| `CHROMIUM_START_URL` | `https://www.tradingview.com/` | URL opened when the browser launches. |
| `CHROMIUM_PROFILE_DIR` | `./chromium-profile` | Browser profile directory. Stores cookies, localStorage, session data. Keep this secure. |
| `CHROMIUM_LOG_FILE_DIR` | `logs` | Directory for browser log files. |
| `CHROMIUM_CRASH_DUMP_DIR` | `logs/chromium-crash-dumps` | Directory for crash dumps. Derived from `CHROMIUM_LOG_FILE_DIR` if not set. |
| `CHROMIUM_ENABLE_CRASH_REPORTER` | `false` | Enable Chromium's crash reporter flags. Keep `false` on sandboxed Linux to avoid Crashpad issues. |

## Controller Settings

These configure the tv_controller REST API server.

| Variable | Default | Description |
|---|---|---|
| `CONTROLLER_BIND_ADDR` | `127.0.0.1:8188` | Address and port the HTTP server binds to. Always use `127.0.0.1`. |
| `CONTROLLER_LAUNCH_BROWSER` | `false` | When `true`, the controller spawns Chromium itself (single-command startup). |
| `CONTROLLER_TAB_URL_FILTER` | `tradingview.com` | Only operate on browser tabs whose URL contains this string. |
| `CONTROLLER_EVAL_TIMEOUT_MS` | `5000` | Timeout in milliseconds for each JavaScript evaluation in the browser. Minimum 1000. |
| `CONTROLLER_LOG_LEVEL` | `info` | Logging level: `debug`, `info`, `warn`, `error`. |
| `CONTROLLER_LOG_FILE` | `logs/tv_controller.log` | Path to the controller's log file. Rotated at 25MB, 10 backups. |
| `SNAPSHOT_DIR` | `./snapshots` | Directory for chart screenshot storage. |

## Researcher Settings

These configure the passive traffic capture daemon (a separate binary from the controller).

| Variable | Default | Description |
|---|---|---|
| `RESEARCHER_DATA_DIR` | `./research_data` | Base directory for captured traffic. Organized by date. |
| `RESEARCHER_TAB_URL_FILTER` | `tradingview.com` | Only attach to tabs matching this URL filter. |
| `RESEARCHER_RELOAD_ON_ATTACH` | `true` | Reload tabs after attaching to capture initial resources. |
| `RESEARCHER_MAX_FILE_SIZE_MB` | `200` | JSONL file rotation size in MB. |
| `RESEARCHER_BUFFER_SIZE` | `5000` | Async writer channel buffer size. |
| `RESEARCHER_CAPTURE_HTTP` | `true` | Capture HTTP request/response pairs. |
| `RESEARCHER_CAPTURE_WS` | `true` | Capture WebSocket frames. |
| `RESEARCHER_CAPTURE_STATIC` | `true` | Capture static resources (JS, CSS, images). |
| `RESEARCHER_HTTP_MAX_BODY_BYTES` | `52428800` | Maximum HTTP body size to capture (50MB). |
| `RESEARCHER_WS_MAX_FRAME_BYTES` | `20971520` | Maximum WebSocket frame size to capture (20MB). |
| `RESEARCHER_RESOURCE_MAX_BYTES` | `104857600` | Maximum static resource size to capture (100MB). |

## Example .env

```bash
# CDP
CHROMIUM_CDP_ADDRESS=127.0.0.1
CHROMIUM_CDP_PORT=9220
CHROMIUM_START_URL=https://www.tradingview.com/
CHROMIUM_PROFILE_DIR=./chromium-profile

# Controller
CONTROLLER_BIND_ADDR=127.0.0.1:8188
CONTROLLER_LAUNCH_BROWSER=true
CONTROLLER_TAB_URL_FILTER=tradingview.com
CONTROLLER_EVAL_TIMEOUT_MS=5000
CONTROLLER_LOG_LEVEL=info
CONTROLLER_LOG_FILE=logs/tv_controller.log
```
