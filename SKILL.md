---
name: claude-browser-traffic
description: "Reverse engineer APIs by capturing browser traffic with mitmproxy. Use when the user says \"capture browser traffic\" or \"reverse engineer API\". The user browses their own browser via mitmproxy + a proxy switcher extension (FoxyProxy on Firefox, TabProxy on Chromium)."
user-invocable: true
allowed-tools: Bash, Read, Grep, Glob
argument-hint: "[proxy|stop|parse] [name-or-har-path]"
---

# API Reverse Engineering Toolkit

Capture traffic with mitmproxy while the user drives their own browser. Best for authenticated flows — uses the user's existing browser sessions/cookies.

## Capture

Run `${CLAUDE_SKILL_DIR}/bcap proxy --name "<name>"` as a background task. Tell the user to switch their proxy switcher to "mitmproxy" and browse. When they're done, run `${CLAUDE_SKILL_DIR}/bcap stop` to stop the proxy and flush the HAR. Then run `${CLAUDE_SKILL_DIR}/bcap parse <har-path>`.

### Port conflicts

Default port is 9090. If mitmdump fails to bind (port in use), tell the user 9090 was taken and retry with `--port 9091`, then 9092, etc., up to 9099. If 9090–9099 all fail, stop and tell the user — do not keep trying.

## After capture

Parse writes a flat NDJSON file (`<name>-entries.ndjson`), one row per non-static request. Query it with `bcap query` — do not read the file directly, it can be large and contains raw bodies/headers/credentials.

```
${CLAUDE_SKILL_DIR}/bcap query captures/<file>-entries.ndjson "SELECT method, host, pattern, status FROM entries WHERE host = '...' LIMIT 20"
```

The NDJSON rows are exposed as a view named `entries`. Columns: `method, url, host, path, pattern, query, status, mime, started_at, duration_ms, request_headers, response_headers, request_body, response_body`.

- `pattern` is `path` with numeric/UUID/hex segments rewritten to `:id` — useful for `GROUP BY pattern` to collapse polling/pagination.
- `query`, `request_headers`, `response_headers` are JSON objects (header names lowercased). Bodies are raw text — use DuckDB's `->>` to extract fields, e.g. `response_body::JSON ->> 'id'`.
- Start with `SELECT host, pattern, COUNT(*) ... GROUP BY 1,2 ORDER BY 3 DESC` to see the shape of the session, then drill into specific endpoints. Summarize auth mechanism, base URL, endpoints, pagination. Ask which endpoints the user needs.

## Integrate

Query `${CLAUDE_SKILL_DIR}/captures/<name>-entries.ndjson` with `bcap query` to pull the exact request shapes you need. If asked to integrate an API, write integration code directly in the current project using its language/framework. Use the exact auth pattern, headers, and request shapes from the capture. Write purpose-built functions, not a generic SDK.

## Save API spec

When the user says "save a spec" after a capture/analysis, write a concise API spec to `${CLAUDE_SKILL_DIR}/specs/<name>.md` with: base URL, auth pattern, and for each endpoint: method, path pattern, params, request/response shape. Keep it machine-readable (YAML frontmatter + markdown tables). Future sessions can read these specs to implement integrations without recapturing. **NEVER include sensitive data in specs** — strip all API keys, tokens, session IDs, passwords, cookies, and other secrets. Describe the auth *mechanism* (e.g. "Bearer token in Authorization header") but never include actual credential values.
