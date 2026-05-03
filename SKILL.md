---
name: claude-browser-traffic
description: "Reverse engineer APIs by capturing browser traffic with mitmproxy. Use when the user says \"capture API traffic\" or \"reverse engineer API\". The user browses their own browser via mitmproxy + a proxy switcher extension (FoxyProxy on Firefox, TabProxy on Chromium)."
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

Read the parsed JSON and summarize: auth mechanism, base URL, endpoints, pagination, and notable headers. Ask which endpoints they need.

## Integrate

Look in `${CLAUDE_SKILL_DIR}/captures/` for the relevant parsed JSON. Write integration code directly in the current project using its language/framework. Use the exact auth pattern, headers, and request shapes from the capture. Write purpose-built functions, not a generic SDK.

## Save API spec

When the user says "write that down" after a capture/analysis, write a concise API spec to `${CLAUDE_SKILL_DIR}/specs/<name>.md` with: base URL, auth pattern, and for each endpoint: method, path pattern, params, request/response shape. Keep it machine-readable (YAML frontmatter + markdown tables). Future sessions can read these specs to implement integrations without recapturing. **NEVER include sensitive data in specs** — strip all API keys, tokens, session IDs, passwords, cookies, and other secrets. Describe the auth *mechanism* (e.g. "Bearer token in Authorization header") but never include actual credential values.
