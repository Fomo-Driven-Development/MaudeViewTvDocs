# Building Agents

The core gives you 177 REST endpoints. An agent picks the ones it needs and wraps them as tools. This page shows the pattern using Go and the MCP SDK, but any language that can make HTTP requests works.

## The Pattern

Every MCP tool follows the same structure:

1. Define an input struct
2. Register the tool with a name and description
3. In the handler, make an HTTP call to tv_controller
4. Return the response

### Helper Functions

Two small helpers handle all the HTTP and result formatting:

```go
// doAPI makes an HTTP request to the tv_controller.
func doAPI(ctx context.Context, client *http.Client, baseURL, method, path string, body any) (json.RawMessage, error) {
    var bodyReader io.Reader
    if body != nil {
        data, err := json.Marshal(body)
        if err != nil {
            return nil, fmt.Errorf("marshal request body: %w", err)
        }
        bodyReader = bytes.NewReader(data)
    }

    req, err := http.NewRequestWithContext(ctx, method, baseURL+path, bodyReader)
    if err != nil {
        return nil, fmt.Errorf("create request: %w", err)
    }
    if body != nil {
        req.Header.Set("Content-Type", "application/json")
    }

    resp, err := client.Do(req)
    if err != nil {
        return nil, fmt.Errorf("request %s %s: %w", method, path, err)
    }
    defer resp.Body.Close()

    respBody, err := io.ReadAll(resp.Body)
    if err != nil {
        return nil, fmt.Errorf("read response: %w", err)
    }

    if resp.StatusCode < 200 || resp.StatusCode >= 300 {
        return nil, fmt.Errorf("%s %s returned %d: %s", method, path, resp.StatusCode, string(respBody))
    }

    return json.RawMessage(respBody), nil
}

func textResult(data json.RawMessage) *mcp.CallToolResult {
    return &mcp.CallToolResult{
        Content: []mcp.Content{&mcp.TextContent{Text: string(data)}},
    }
}

func errResult(err error) *mcp.CallToolResult {
    return &mcp.CallToolResult{
        Content: []mcp.Content{&mcp.TextContent{Text: err.Error()}},
        IsError: true,
    }
}
```

### Simple Tool (No Input)

A tool that just calls a GET endpoint:

```go
type GetActiveChartInput struct{}

mcp.AddTool(server, &mcp.Tool{
    Name:        "get_active_chart",
    Description: "Get info about the currently active chart",
}, func(ctx context.Context, req *mcp.CallToolRequest, _ GetActiveChartInput) (*mcp.CallToolResult, any, error) {
    body, err := doAPI(ctx, client, baseURL, http.MethodGet, "/api/v1/charts/active", nil)
    if err != nil {
        return errResult(err), nil, nil
    }
    return textResult(body), nil, nil
})
```

### Tool with Input

A tool that takes parameters and sends them as JSON:

```go
type SetSymbolInput struct {
    Symbol string `json:"symbol" jsonschema:"The ticker symbol (e.g. AAPL, BTCUSD)"`
}

mcp.AddTool(server, &mcp.Tool{
    Name:        "set_symbol",
    Description: "Change the chart symbol",
}, func(ctx context.Context, req *mcp.CallToolRequest, args SetSymbolInput) (*mcp.CallToolResult, any, error) {
    payload := map[string]string{"symbol": args.Symbol}
    body, err := doAPI(ctx, client, baseURL, http.MethodPut, "/api/v1/chart/active/symbol", payload)
    if err != nil {
        return errResult(err), nil, nil
    }
    return textResult(body), nil, nil
})
```

### Composite Tool

A tool that makes multiple API calls to accomplish a higher-level task:

```go
type ChartSetupInput struct {
    Symbol     string `json:"symbol" jsonschema:"Ticker symbol"`
    Resolution string `json:"resolution" jsonschema:"Timeframe (1, 5, 15, 60, D, W, M)"`
    ChartType  string `json:"chart_type" jsonschema:"Chart style (candles, line, area, etc.)"`
}

mcp.AddTool(server, &mcp.Tool{
    Name:        "setup_chart",
    Description: "Set symbol, resolution, and chart type in one call",
}, func(ctx context.Context, req *mcp.CallToolRequest, args ChartSetupInput) (*mcp.CallToolResult, any, error) {
    // Set symbol
    _, err := doAPI(ctx, client, baseURL, http.MethodPut, "/api/v1/chart/active/symbol",
        map[string]string{"symbol": args.Symbol})
    if err != nil {
        return errResult(fmt.Errorf("set symbol: %w", err)), nil, nil
    }

    // Set resolution
    _, err = doAPI(ctx, client, baseURL, http.MethodPut, "/api/v1/chart/active/resolution",
        map[string]string{"resolution": args.Resolution})
    if err != nil {
        return errResult(fmt.Errorf("set resolution: %w", err)), nil, nil
    }

    // Set chart type
    _, err = doAPI(ctx, client, baseURL, http.MethodPut, "/api/v1/chart/active/chart-type",
        map[string]string{"chart_type": args.ChartType})
    if err != nil {
        return errResult(fmt.Errorf("set chart type: %w", err)), nil, nil
    }

    return textResult(json.RawMessage(`{"status":"configured"}`)), nil, nil
})
```

## Steps to Build Your Own

1. **Browse the [API Reference](api-reference.md)** — find the endpoints you need
2. **Fork or copy the [WatchlistManagerAgent](https://github.com/Fomo-Driven-Development/MaudeViewWatchlistManagerAgent)** as a starting point
3. **Delete tools you don't need** — strip it down to your use case
4. **Add your tools** using the patterns above
5. **Install the controller** — `just install-controller` in your agent repo
6. **Run** — start the controller, then your agent

## Project Setup

A minimal `go.mod`:

```
module github.com/yourname/my-tradingview-agent

go 1.24.0

require (
    github.com/joho/godotenv v1.5.1
    github.com/modelcontextprotocol/go-sdk v1.3.0
)
```

A minimal `.env`:

```
CONTROLLER_BIND_ADDR=127.0.0.1:8188
```

A `justfile` with controller install:

```
install-controller:
    GOBIN={{justfile_directory()}}/bin go install github.com/dgnsrekt/MaudeViewTVCore/cmd/tv_controller@latest

run-tv-controller-with-browser:
    CONTROLLER_LAUNCH_BROWSER=true ./bin/tv_controller
```

## Tips

- **Keep tools focused.** A tool called `manage_everything` is less useful than `add_to_watchlist` and `remove_from_watchlist`.
- **Composite tools are powerful.** An agent that sets up a full chart layout (symbol + resolution + indicators + timeframe) in one call is more useful than exposing each operation separately.
- **Use the `jsonschema` tag** on input struct fields to give the LLM clear parameter descriptions.
- **Error messages matter.** The LLM reads error responses to decide what to do next.
