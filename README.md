# Headless Experience Layer — Case Next Action

Public sample of Salesforce [Headless Experience Layer (HXL)](https://www.salesforce.com/headless/agentic-experience-layer/): one Mosaic JSON card, rendered in Claude Desktop via a local MCP tool.

The [blog](docs/blog/20260813-headless-experience-layer.html) is the story. This README is how you run the sample on **your** machine.

The card uses fictional Northwind data (`00001234`). Claude Desktop uses **your** signed-in account. A Salesforce org login is optional and stays on your laptop.

![Case Next Action card](https://raw.githubusercontent.com/Lakshmikanth-Paruchuru/hxl-agentic-experience-layer/main/docs/media/case-next-action.png)

## Setup (short)

You need [Node.js](https://nodejs.org/) 18+ and [Claude Desktop](https://claude.ai/download) (the chat app, not Claude Code). Sign in to Desktop with your own Anthropic account. We do not provide a key.

```bash
git clone https://github.com/Lakshmikanth-Paruchuru/hxl-agentic-experience-layer.git
cd hxl-agentic-experience-layer
cd mcp && npm install && cd ..
node scripts/print-claude-config.mjs
```

The last command prints a `mcpServers` block with **your** absolute `node` path and **your** `mcp/server.mjs` path. Merge that block into Claude Desktop’s config (keep any other servers you already have):

| OS | Config file |
|---|---|
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| Linux | `~/.config/Claude/claude_desktop_config.json` |

Example shape (do not copy the paths — use the script output):

```json
{
  "mcpServers": {
    "hxl-case-next-action": {
      "command": "/absolute/path/to/node",
      "args": ["/absolute/path/to/hxl-agentic-experience-layer/mcp/server.mjs"]
    }
  }
}
```

Then:

1. **Fully quit Claude** (macOS: ⌘Q). Closing the window is not enough.
2. Reopen Claude Desktop. **Settings** (`⌘,`) → **Desktop app** → **Developer** → **Local MCP servers**.
3. Confirm `hxl-case-next-action` shows **running**.
4. New chat, send: `Show me the Case Next Action card`
5. Allow the tool if asked.

![Enable the MCP in Claude Desktop Developer settings](https://raw.githubusercontent.com/Lakshmikanth-Paruchuru/hxl-agentic-experience-layer/main/docs/media/claude-enable-mcp-developer.png)

*Settings → Desktop app → Developer → Local MCP servers. The badge must say **running**. Machine paths in this shot are covered.*

A filled `.env` is **not** required. Copy `.env.example` only if you later add your own org credentials — never commit that file.

## See it in Claude

After the server is **running**, a new chat gets one line:

```
Show me the Case Next Action card
```

You want the white card — not a paragraph with a spinner underneath.

[Watch the Claude Desktop demo](https://github.com/Lakshmikanth-Paruchuru/hxl-agentic-experience-layer/blob/main/docs/media/claude-chat-demo.mp4)

[![Case Next Action in Claude Desktop — click to play the demo](https://raw.githubusercontent.com/Lakshmikanth-Paruchuru/hxl-agentic-experience-layer/main/docs/media/claude-chat-case-next-action.png)](https://github.com/Lakshmikanth-Paruchuru/hxl-agentic-experience-layer/blob/main/docs/media/claude-chat-demo.mp4)

*Identity in the sidebar is blurred. Sample data is fictional (Northwind Traders, case `00001234`). Buttons send a chat message; they do not write to Salesforce.*

## Try the playground (no install)

1. Open the [HXL Playground](https://www.headlessexperiencelayer.com/playground/).
2. Paste `mosaics/case-next-action.json`.
3. Switch Slack / ChatGPT / Claude / Agentforce previews.

Local semantic preview:

```bash
python3 -m http.server 8766
# http://127.0.0.1:8766/preview/
```

## What you need to know

| Term | Meaning |
|---|---|
| **HXL** | Describe a UI once; each host paints it. |
| **Mosaic** | Playground JSON (`{!$case.field}`). |
| **WidgetBundle** | Org copy (`{!$attrs.field}` + `schema.json`). Optional for Claude. |
| **MCP** | Local tool Claude starts over **stdio**. |
| **Claude Desktop** | Chat app. Not Claude Code. |

## What is in this repo

| Path | Role |
|---|---|
| `mosaics/case-next-action.json` | Playground Mosaic |
| `force-app/main/default/uiWidgets/caseNextAction/` | WidgetBundle (org; do not deploy unless you intend to) |
| `mcp/server.mjs` | MCP server — tool `get_case_next_action` |
| `mcp/widgets/case-next-action.html` | HTML Claude iframes |
| `mcp/claude-mcp.example.json` | Config shape only — no machine paths |
| `scripts/print-claude-config.mjs` | Prints *your* Desktop config |
| `docs/blog/` | Public post |

## How the sample is built

1. **HXL mosaic** — tiles + `{!$case.field}` + `meta.if` for the SLA callout. When `slaAtRisk` is false the callout is not in the tree.
2. **WidgetBundle** — same tiles, `{!$attrs.*}`, `schema.json`. Validate with `python3 scripts/validate-widget.py`.
3. **MCP app** — `npm install` in `mcp/`. Server speaks stdio, inlines `@modelcontextprotocol/ext-apps`, advertises `ui://widgets/case-next-action.html`. Do not put `mcp-remote` / HTTP in front of Desktop.
4. **Enable in Desktop** — Setup above. The Developer screenshot is in that section.
5. **Use it** — `Show me the Case Next Action card`. See the video and still in **See it in Claude**. Buttons send a chat message; they do not write to Salesforce.

## If you only see a spinner

| Cause | Fix |
|---|---|
| Handshake bundle missing | Keep the inlined `ext-apps` in `server.mjs`. No CDN. |
| HTTP + `mcp-remote` | Native stdio only. |
| Stale chat | Quit fully, new chat. |
| Server not running | Developer page must say **running**. Wrong path or `node` not found. |

## Secrets and org access

- This repo has **no** Salesforce Connected App, Anthropic key, or customer data.
- Claude: use your own Desktop login. Never paste a session token into git.
- Live Case data later: `sf org login web` on your machine, or put Connected App values in a local `.env` from `.env.example`. `.sfdx/`, `.env`, and `credentials.json` are gitignored.
- Do not deploy the WidgetBundle to an org unless you mean to, and only to **your** org.

## License

MIT. Product names belong to Salesforce, Inc. and Anthropic. Independent sample, not an official Salesforce or Anthropic repository.
