# Mynahcast MCP server

**Mynahcast** is an agent-first social media scheduler. The AI agent you already use (Claude, Claude Code, ChatGPT, Cursor, Codex, Gemini and others) drafts, validates and schedules posts in your brand's voice. A person approves every agent post before it goes out, and Mynahcast reads each post back from the platform to confirm it is live where the platform allows it.

This repository documents Mynahcast's **official remote MCP server**. It holds configuration and docs only. The server is hosted by Mynahcast; there is nothing to install or self-host.

- Website: https://mynahcast.xyz
- MCP docs: https://mynahcast.xyz/docs/mcp
- Official MCP Registry name: `xyz.mynahcast/mynahcast`

## Endpoint

| | |
|---|---|
| URL | `https://api.mynahcast.xyz/mcp` |
| Transport | Streamable HTTP, stateless JSON-RPC 2.0 (`initialize`, `tools/list`, `tools/call`) |
| Auth | `Authorization: Bearer <Mynahcast API key>` on every request |
| Discovery | `https://mynahcast.xyz/.well-known/mcp/server-card.json` |

Create an API key in the Mynahcast dashboard under **Agents & API → Create key**. Pick the **Agent** preset so your agent's posts wait in the Approvals queue. The key is shown once. Keys go in the header, never in the URL.

## Quick setup

**Claude Code**
```bash
claude mcp add --transport http mynahcast https://api.mynahcast.xyz/mcp \
  --header "Authorization: Bearer $MYNAHCAST_KEY"
```

**Cursor** (and other clients that read an `mcpServers` file)
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

**Claude Desktop**, **ChatGPT** and more: see [docs/setup.md](docs/setup.md).

## What your agent can do

Tools are named `mynahcast_*` and filtered by the key's scopes. Every tool that changes data accepts `dry_run`.

- Validate a post against every target channel's limits before saving it
- Create, edit, reschedule and delete posts, fanned out to one or more channels
- Submit posts for approval (agent keys can't approve; a person does)
- Read a post's delivery state and full timeline of attempts, approvals and read-backs
- List connected channels and each channel's limits
- Upload media through signed URLs
- Read the brand voice profile, product context and saved styles before writing

Each post is delivered once per channel (writes take an `Idempotency-Key`), so a retry never double-posts.

## Live channels (13)

Bluesky, Mastodon, Telegram, Discord, Nostr, Lemmy, Misskey, Slack, Matrix, DEV, Hashnode, WordPress, Listmonk.

## Pricing

MCP and REST API access is included on every plan: Creator $12, Pro $24, Team $39, Agency $79 per month per workspace. Every plan starts with a 7-day free trial, no card needed. See https://mynahcast.xyz/pricing.

## Support

support@mynahcast.xyz

The Mynahcast service itself is proprietary. The files in this repository are MIT licensed.
