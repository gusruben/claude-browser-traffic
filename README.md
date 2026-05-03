# api-re

Claude Code skill for reverse engineering APIs by capturing browser traffic with mitmproxy.

## Setup

Clone into your Claude skills directory:

```bash
git clone https://github.com/gusruben/claude-browser-traffic "$HOME/.claude/skills/api-re"
```

(`$HOME` works in bash, zsh, and PowerShell. On `cmd.exe`, use `%USERPROFILE%` instead.)

Add to `~/.claude/settings.json`:
```json
{
  "permissions": {
    "allow": [
      "Read(~/.claude/skills/api-re/captures/**)",
      "Bash(~/.claude/skills/api-re/api-re:*)"
    ],
    "additionalDirectories": ["~/.claude/skills/api-re"]
  }
}
```

### Install mitmproxy

See the [installation guide](https://docs.mitmproxy.org/stable/overview/installation/) — works on Linux, macOS, and Windows.

### Install a proxy switcher

Pick one based on your browser:

- **Chrome / Edge / Brave / other Chromium**: [TabProxy](https://chromewebstore.google.com/detail/tabproxy-%E2%80%94-per-site-per-t/lgkgojfkajhkhohpgnpmmkodmbdkaiea)
- **Firefox**: [FoxyProxy Standard](https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard/)

Add a proxy entry: HTTP, host `127.0.0.1`, port `9090`, name it `mitmproxy`.

### Trust the mitmproxy CA cert

1. Run `mitmdump -p 9090` in a terminal
2. Switch your browser to the `mitmproxy` proxy and open <http://mitm.it> — mitmproxy serves the cert plus per-OS install instructions (Linux, macOS, Windows, iOS, Android)
3. Follow the instructions for your OS + browser, then stop `mitmdump`

Switch the proxy back off when not capturing.

## Usage

Tell Claude: "capture API traffic" or "reverse engineer API" — or run `/api-re`.

Claude starts mitmproxy, you flip the browser proxy to `mitmproxy` and browse, then come back when done. After capture, say "write that down" to save a reusable API spec.
