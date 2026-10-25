# Connecting an agent to Mynahcast

## 1. Create an API key

In Mynahcast, open **Agents & API → Create key** and choose the **Agent** preset. Agent keys can draft, validate and schedule posts, but the server refuses approval decisions from them, so every post waits for a person in **Approvals**. Copy the key (it is shown once) and keep it out of shared files and URLs.

## 2. Add the server to your client

Endpoint: `https://api.mynahcast.xyz/mcp` (streamable HTTP). Header: `Authorization: Bearer <key>`.

### Claude Code
```bash
claude mcp add --transport http mynahcast https://api.mynahcast.xyz/mcp \
  --header "Authorization: Bearer $MYNAHCAST_KEY"
```

### Claude Desktop
Settings → Developer → Edit config, then add the server to `claude_desktop_config.json`. This uses the open-source [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) bridge (needs Node.js) to send the key as a header:
```json
{
  "mcpServers": {
    "mynahcast": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://api.mynahcast.xyz/mcp", "--header", "Authorization:${MYNAHCAST_AUTH}"],
      "env": { "MYNAHCAST_AUTH": "Bearer myn_live_..." }
    }
  }
}
```
Restart Claude Desktop; the Mynahcast tools appear in the tools menu.

### Cursor
Add to `~/.cursor/mcp.json` (or the project's `.cursor/mcp.json`):
```json
{
  "mcpServers": {
    "mynahcast": {
      "url": "https://api.mynahcast.xyz/mcp",
      "headers": { "Authorization": "Bearer myn_live_..." }
    }
  }
}
```

### ChatGPT (custom GPT actions)
ChatGPT connects through the REST API, which exposes the same operations and approval rules as the MCP server.
1. In ChatGPT, create a GPT and open **Configure → Create new action**.
2. Choose **Import from URL** and paste `https://api.mynahcast.xyz/v1/openapi.json`.
3. Under **Authentication** pick **API Key**, Auth Type **Bearer**, and paste your Mynahcast key.
4. Save, then ask the GPT to draft a post. It lands in your approval queue.

### Other clients
Codex, Gemini CLI, OpenClaw, Hermes Agent, Kimi, Manus and others: guides at https://mynahcast.xyz/docs/agents.

## 3. Check the connection
```bash
curl -X POST https://api.mynahcast.xyz/mcp \
  -H "Authorization: Bearer $MYNAHCAST_KEY" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```
Without a key the server answers `401` with `WWW-Authenticate: Bearer realm="mynahcast"`.

Then ask your agent for a post: it validates against each channel, saves a draft and submits it for approval. You approve it in the app, and Mynahcast publishes and reads it back.

More: https://mynahcast.xyz/docs/mcp
