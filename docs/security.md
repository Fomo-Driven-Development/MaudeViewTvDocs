# Security

MaudeView connects to a live Chromium browser session via Chrome DevTools Protocol with full privileges — JavaScript execution, cookie access, network interception, and DOM manipulation. Neither the CDP port nor the controller REST API use authentication. Treat the dedicated browser profile, all listening ports, and captured data as sensitive.

## Browser Profile

The `chromium-profile/` directory stores session cookies, localStorage, IndexedDB, and autofill data in plaintext. Anyone who copies this directory has your TradingView session — no password required.

**Rule**: Only log into TradingView in this browser. Never log into email, banking, social media, or any other service.

- Restrict directory permissions: `chmod 700 chromium-profile/`
- Never commit it to git
- Exclude from cloud sync (Dropbox, Google Drive, OneDrive)
- Delete the directory when you no longer need the session

## CDP Port

CDP provides unrestricted browser control to anyone who can reach the port. A connection to the CDP port can execute arbitrary JavaScript, read cookies and localStorage, intercept all network traffic, and take screenshots.

**Rule**: Always bind to `127.0.0.1`, never `0.0.0.0`.

- Verify `CHROMIUM_CDP_ADDRESS=127.0.0.1` in your `.env`
- Firewall the CDP port (default `9220`)
- Do not run on shared machines or a VPS without a firewall
- Stop the browser when not actively using it

## Controller API

The REST API at `CONTROLLER_BIND_ADDR` (default `127.0.0.1:8188`) has no authentication. It can change symbols, create and delete alerts, modify watchlists, execute Pine scripts, and take chart snapshots.

**Rule**: Always bind to `127.0.0.1`, never expose to the network.

- Verify `CONTROLLER_BIND_ADDR=127.0.0.1:8188` in your `.env`
- Firewall the port if running on a multi-user system
- Stop the controller when not in use

## Research Data

The researcher captures full HTTP request/response bodies, including authorization headers, cookies, and session tokens. These are written as JSONL files to `research_data/`.

**Rule**: Treat research data as confidential.

- Restrict directory permissions: `chmod 700 research_data/`
- Purge old captures regularly
- Review files before sharing any extracts

## Snapshots

Chart screenshots may contain visible trading positions, P&L, alert configurations, and strategy details.

**Rule**: Review snapshots before sharing.

- Restrict directory permissions: `chmod 700 snapshots/`
- Delete snapshots you no longer need

## Quick Checklist

| Do | Don't |
|---|---|
| Bind CDP and controller to `127.0.0.1` | Bind to `0.0.0.0` or a public interface |
| Only log into TradingView in the dedicated browser | Log into email, banking, or other services |
| `chmod 700` on `chromium-profile/`, `research_data/`, `snapshots/` | Leave directories world-readable |
| Stop the browser and controller when not in use | Leave them running unattended on shared machines |
| Firewall CDP and controller ports | Expose ports on a VPS without a firewall |
| Purge old research data and snapshots | Accumulate captures indefinitely |
| Keep `.env` out of version control | Commit `.env` or `chromium-profile/` to git |
