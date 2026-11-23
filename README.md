<p align="center"><img src="banner.jpg" alt="netloop.ai" width="720"></p>

# netloop.ai — MCP server

[netloop.ai](https://netloop.ai) is a **CLI-based assistant for network operations**,
exposed as a remote **Model Context Protocol (MCP)** server and as an
[A2A](https://a2a.netloop.ai/.well-known/agent-card.json) agent.

Ask questions about switches, routers, access points and firewalls in plain
language — the platform-appropriate CLI command is chosen for you.

- **Read-only by default.** Write operations require `read_only: false` in the
  connector configuration and an explicit **Human-in-the-Loop** approval.
- **Cross-vendor.** ArubaOS / AOS-CX / AOS-8 / Aruba Instant, Cisco IOS / NX-OS,
  Juniper Junos, Comware, FortiOS.
- **EU-hosted, GDPR.** Processing in the EU; no training on your data.

## Endpoints

| Transport | URL |
|---|---|
| Streamable HTTP | `https://mcp.netloop.ai/mcp` |
| SSE (deprecated) | `https://mcp.netloop.ai/sse` |

Discovery documents: [`/.well-known/agent-card.json`](https://a2a.netloop.ai/.well-known/agent-card.json)
(A2A), [`llms.txt`](https://netloop.ai/llms.txt), [`agents.md`](https://netloop.ai/agents.md),
[`/.well-known/ai-catalog.json`](https://netloop.ai/.well-known/ai-catalog.json) (ARD).

Published in the official MCP registry as **`ai.netloop/mcp`**:
`https://registry.modelcontextprotocol.io/v0.1/servers?search=ai.netloop/mcp`

## Authentication

Requests carry the tenant token in the **`X-Auth-Token`** header
(identifies the tenant; device credentials are delivered separately,
end-to-end encrypted). Discovery calls (`initialize`, `tools/list`,
`resources/list`, `prompts/list`) answer without a token.

```bash
curl -sS -X POST https://mcp.netloop.ai/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

<!-- MCP authorization (OAuth 2.1) is in progress: see
     https://netloop.ai/.well-known/oauth-protected-resource (planned). -->

## Quick start

**VS Code / Cline**

```json
{
  "mcp": {
    "servers": {
      "netloop": {
        "type": "http",
        "url": "https://mcp.netloop.ai/mcp",
        "headers": { "X-Auth-Token": "YOUR_TOKEN" }
      }
    }
  }
}
```

**Cursor** (`~/.cursor/mcp.json`)

```json
{
  "mcpServers": {
    "netloop": {
      "url": "https://mcp.netloop.ai/mcp",
      "headers": { "X-Auth-Token": "YOUR_TOKEN" }
    }
  }
}
```

**Claude Code**

```bash
claude mcp add --transport http netloop https://mcp.netloop.ai/mcp \
  --header "X-Auth-Token: YOUR_TOKEN"
```

**Any stdio-only client** (e.g. Claude Desktop) can bridge to the remote server
with [`mcp-remote`](https://www.npmjs.com/package/mcp-remote):

```json
{
  "mcpServers": {
    "netloop": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.netloop.ai/mcp",
               "--header", "X-Auth-Token:${NETLOOP_TOKEN}"]
    }
  }
}
```

## Tools

All tools ship MCP [annotations](https://modelcontextprotocol.io/) so clients can
show badges and gate writes correctly. `R` = read-only, `W` = can change a device
(always gated by Human-in-the-Loop approval).

| Tool | Title | Hints |
|---|---|---|
| `identify_platform` | Identify platform | R, open-world |
| `netloop_ask` | Ask a device | **W**, destructive, open-world |
| `netloop_fetch` | Fetch device output | **W**, destructive, open-world |
| `netloop_explain` | Explain device output | R, open-world |
| `netloop_path` | Show network path | R, open-world |
| `netloop_vuln` | Network vulnerability check | R, open-world |
| `netloop_drill` | Explore CLI help | R, open-world |
| `netloop_recover_change` | Recover a rejected change | R, open-world |
| `netloop_confirm_change` | Confirm configuration change | **W**, destructive, open-world |
| `netloop_submit_credential` | Submit device credential | session |
| `netloop_support_grant` | Grant support access | session (admin) |
| `vendor_lookup` | Look up MAC vendor | R, local |
| `acronym_lookup` | Expand an acronym | R, local |
| `skill_lookup` | Search documentation | R, local |
| `netloop_commands` | Look up documented commands | R, local |

## Links

- Website: <https://netloop.ai>
- Agent guide (setup, enrolment, egress): <https://netloop.ai/agents.md>
- Open source / licences: <https://netloop.ai/en/open-source-licenses>
- Privacy: <https://netloop.ai/en/privacy> · Terms: <https://netloop.ai/en/terms>
- Contact: <human@netloop.ai>

## About this repository

This repository only carries the **server metadata** (`server.json`) and the
integration notes above — the MCP server itself is a hosted service; the
connection to the network devices is made by the netloop.ai connector that runs
inside your network. No credentials, tokens or customer data are stored here.
