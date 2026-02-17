# API Reference

The tv_controller exposes 177 REST endpoints for TradingView automation. Browse the full interactive reference below — every endpoint, request/response schema, and parameter is documented.

<div style="height: 85vh;" id="api-reference">
  <script
    id="api-reference-script"
    data-url="./openapi.json"
    data-configuration='{"theme":"deepSpace","hiddenClients":true}'>
  </script>
  <script src="https://cdn.jsdelivr.net/npm/@scalar/api-reference"></script>
</div>

## Endpoint Overview

| Feature Area | Endpoints | Description |
|---|---|---|
| Charts | 28 | Tab listing, active chart, symbol, resolution, chart type |
| Watchlists | 15 | CRUD, symbol management, colored lists |
| Drawings | 20 | Lines, shapes, text, tweets, z-order, visibility |
| Studies & Indicators | 15 | Add/remove/configure studies, search, favorites |
| Layouts | 19 | Save, rename, clone, delete, grid templates, multi-pane |
| Pine Editor | 21 | Toggle, source read/write, save, add to chart, console |
| Replay | 14 | Activate, step, autoplay, date navigation |
| Alerts | 14 | CRUD, start/stop, fires, clone |
| Misc | 28 | Health, strategy, snapshots, currency, units, hotlists |

## Base URL

When running locally, the default base URL is:

```
http://127.0.0.1:8188
```

All endpoints are prefixed with `/api/v1/` except for `/health` and `/docs`.
