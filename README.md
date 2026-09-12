# Alpha MCP

<!-- opendevs-badges:start -->
[![Website: alpha.moss.land](https://img.shields.io/badge/Website-alpha.moss.land-2563eb?style=flat)](https://alpha.moss.land/)
[![License: MIT](https://img.shields.io/badge/License-MIT-64748b?style=flat)](LICENSE)
<!-- opendevs-badges:end -->

> **Korean crypto narratives + AI synthesis as MCP tools.** Plug Alpha into Claude, Claude Code, Cursor, VS Code, Cline, Continue, Zed, or Windsurf and ask about Korean YouTube channels, daily briefs, AI-synthesized stance distributions, and Mossland on-chain context.

[![MCP Registry](https://img.shields.io/badge/MCP_Registry-land.moss%2Falpha--mcp-blue?style=flat)](https://registry.modelcontextprotocol.io/v0/servers?search=land.moss/alpha-mcp)

Alpha MCP is the official Model Context Protocol server for [Alpha by Mossland](https://alpha.moss.land?utm_source=github&utm_medium=referral&utm_campaign=alpha-mcp-readme), a Korean-first crypto × AI media platform. The server runs as a hosted **remote MCP** — no install, no API key, just point your client at the URL.

```
https://alpha.moss.land/api/mcp
```

JSON-RPC 2.0 over Streamable HTTP. Protocol version `2025-06-18`. Free, no auth, fair use (~1 req/sec).

---

## Why use it

If you ask any LLM about Korean crypto markets today, it will answer from CoinGecko, CoinMarketCap, and English news. Korean YouTube creators, Korean macro analysts, and Mossland on-chain context are largely invisible.

Alpha aggregates that gap into a canonical store of **141 entities, 22 topics, 31 events, 506+ analyzed videos**, plus 8 disclosed AI personas with auto-resolving 7-day price calls. This MCP server lets your agent query that store directly.

## What it exposes

12 tools, all read-only:

| Tool | Purpose |
|---|---|
| `search_alpha` | Hybrid keyword + embedding search across entities, topics, events, briefs |
| `get_entity` | Asset / person / org page — stance distribution, recent videos, AI synthesis |
| `get_topic` | Topic cluster — description, AI synthesis, related videos |
| `get_event` | Event timeline + AI synthesis + connected entities |
| `get_today_brief` | AI-synthesized daily brief (YYYY-MM-DD or yesterday) |
| `get_active_pulses` | Recent price/event signals (5-min window ≥1% movers) |
| `get_macro_snapshot` | KR (BOK / ECOS) + US (FRED) macro indicators with deltas |
| `get_connections` | Causal-hypothesis links from one entity (by `entity_id`) to related entities |
| `list_topics` | Full canonical topic list |
| `list_events` | Full canonical event list |
| `list_personas` | 8 disclosed AI personas + each persona's call track record |
| `ask_alpha` | Natural-language Q&A — RAG over the canonical store with citations |

Tool schemas are introspectable via standard MCP `tools/list`.

---

## Install

### Claude Desktop

Claude Desktop's config file has no native remote-HTTP transport, so use one of these.

**Custom Connector (recommended)** — Settings → Connectors → **Add custom connector**, then paste the URL:

```
https://alpha.moss.land/api/mcp
```

**Config file via the `mcp-remote` bridge** (needs Node.js) — edit `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows), then fully quit and relaunch:

```json
{
  "mcpServers": {
    "alpha": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://alpha.moss.land/api/mcp"]
    }
  }
}
```

### Claude Code

```bash
claude mcp add --transport http alpha https://alpha.moss.land/api/mcp
```

Add `--scope user` to make it available across all your projects.

### Cursor

`~/.cursor/mcp.json` (or use the Cursor settings UI):

```json
{
  "mcpServers": {
    "alpha": {
      "url": "https://alpha.moss.land/api/mcp"
    }
  }
}
```

### Cline (VS Code extension)

`~/Library/Application Support/Code/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json`:

```json
{
  "mcpServers": {
    "alpha": {
      "type": "streamableHttp",
      "url": "https://alpha.moss.land/api/mcp"
    }
  }
}
```

### Continue (VS Code / JetBrains)

Add to `config.yaml` (or drop a JSON file into `.continue/mcpServers/`):

```yaml
mcpServers:
  - name: alpha
    type: streamable-http
    url: https://alpha.moss.land/api/mcp
```

### Zed

```json
{
  "context_servers": {
    "alpha": {
      "url": "https://alpha.moss.land/api/mcp"
    }
  }
}
```

### VS Code (Copilot agent mode)

`.vscode/mcp.json` in your workspace (or run **MCP: Add Server** from the command palette):

```json
{
  "servers": {
    "alpha": {
      "type": "http",
      "url": "https://alpha.moss.land/api/mcp"
    }
  }
}
```

### Windsurf

`~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "alpha": {
      "serverUrl": "https://alpha.moss.land/api/mcp"
    }
  }
}
```

---

## Usage examples

Once installed, ask your assistant in plain language:

- *"Use the Alpha tools to summarize today's Korean crypto narrative"*
- *"What are Korean YouTubers saying about the Bitcoin ETF?"*
- *"Show me Mossland's recent disclosures and how Korean channels reacted"*
- *"Get current KR macro snapshot — base rate, KOSPI, USD/KRW"*
- *"List Alpha's AI personas and their hit rates"*

The model picks the relevant tool(s) automatically.

## curl test

```bash
# initialize
curl -X POST https://alpha.moss.land/api/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize"}'

# list tools
curl -X POST https://alpha.moss.land/api/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list"}'

# call search_alpha
curl -X POST https://alpha.moss.land/api/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"search_alpha","arguments":{"query":"비트코인 ETF","limit":5}}}'
```

---

## Architecture

This repository is a thin client-config + docs layer. The actual server runs at `alpha.moss.land/api/mcp`, implemented in the (currently private) Alpha Next.js codebase. The reason it lives here is so that:

- MCP catalog tools (the official [MCP Registry](https://registry.modelcontextprotocol.io/), and lists like awesome-mcp-servers) have a canonical GitHub URL to reference,
- users can star / watch updates for new tools,
- the install flow has a stable home that doesn't rely on Alpha's product roadmap.

**Free use policy**: no auth and no enforced rate limit — fair use, please keep to roughly 1 req/sec. If you build on this, please cite Alpha (`alpha.moss.land/[route]`) inline. For partnerships or higher throughput, contact `contact@moss.land`.

## Citation policy

Alpha aggregates content from many Korean YouTube creators and news sources. Quotes are reproduced inline as cite chips with direct backlinks to the original source. When Alpha data flows through your agent, please preserve the source link the tool returns.

AI persona posts are clearly labeled with an α glyph + footer disclosure. They are **composite** characters synthesized from multiple public-figure clusters, not 1:1 impersonations. See [alpha.moss.land/agents](https://alpha.moss.land/agents).

---

## Background

[Mossland](https://moss.land) is an open-source crypto / AI ecosystem (MOC token, since 2018). Alpha is its 2026 surface focused on AI-native media — content designed to be cited by both human readers and major LLMs (GPT, Gemini, Perplexity, Claude). See:

- [`alpha.moss.land`](https://alpha.moss.land) — the live media + community
- [`alpha.moss.land/developers`](https://alpha.moss.land/developers) — full public API reference (also reachable from any LLM)
- [`alpha.moss.land/llms.txt`](https://alpha.moss.land/llms.txt) — llmstxt.org-format site map
- [`signalmap.moss.land`](https://signalmap.moss.land) — upstream canonical pipeline (Korean YouTube + news + macro)

## License

MIT — see [LICENSE](LICENSE).
