---
name: mynahcast
description: Run a brand's social media through Mynahcast. Draft, validate, schedule and publish posts to Bluesky, Mastodon, Telegram, Discord, Nostr, Lemmy, Misskey, Slack, Matrix, DEV, Hashnode, WordPress and Listmonk, in the brand's own voice, with a human approval step. Use when the user asks to post, schedule, plan a week of content, or check how posts went.
homepage: https://mynahcast.xyz
metadata: {"env": ["MYNAHCAST_API_KEY"]}
---

# Mynahcast

Mynahcast publishes posts for a brand across many social channels. You write; a person approves; Mynahcast delivers each post once per channel and reports back.

## Setup

1. The user signs up at https://mynahcast.xyz (7-day free trial) and connects their channels.
2. In the app: **Agents & API → Create key → Agent preset**. Agent keys can draft and schedule but can never approve, so every post waits for a person in **Approvals**.
3. Put the key in the env var `MYNAHCAST_API_KEY`. Never print it, never put it in a URL.

Prefer the MCP server when your client supports remote MCP: `https://api.mynahcast.xyz/mcp` (streamable HTTP, header `Authorization: Bearer $MYNAHCAST_API_KEY`). Otherwise use the REST API below.

## REST basics

- Base URL: `https://api.mynahcast.xyz`
- Auth: `-H "Authorization: Bearer $MYNAHCAST_API_KEY"`
- JSON in, JSON out. Errors are RFC 9457 problem+json: read `detail` and fix the request.
- Writes accept `Idempotency-Key: <uuid>`. Always send one, so a retry never double-posts.
- `?dry_run=true` on create validates and previews without saving.
- Full spec: `GET /v1/openapi.json`

## The workflow

**1. Find the brand and its channels**

```bash
curl -s https://api.mynahcast.xyz/v1/channels -H "Authorization: Bearer $MYNAHCAST_API_KEY"
```
Each item has `id` (channelId), `brandId`, `provider`, `handle`, `status`. Only use channels with `status: "active"`. If one says `needs_reconnect`, tell the user to reconnect it in the app.

**2. Read the brand before writing** (do this every session)

- `GET /v1/brands/{brandId}/voice`: summary, do/don't rules, banned terms, example posts. Follow them.
- `GET /v1/brands/{brandId}/product-context`: what the product is, who it's for, proof, CTA, banned claims. Never make a claim that isn't here.
- `GET /v1/brands/{brandId}/content-memory`: what was already posted or rejected, with reasons. Don't repeat hooks or topics; learn from rejections.
- `GET /v1/brands/{brandId}/styles`: saved post styles; the default comes first.

**3. Check limits for each channel**

`GET /v1/channels/{channelId}/capabilities`: character limits, media rules, settings. Write a per-channel override when a post won't fit.

**4. Validate, then create**

```bash
curl -s -X POST https://api.mynahcast.xyz/v1/posts/validate \
  -H "Authorization: Bearer $MYNAHCAST_API_KEY" -H "Content-Type: application/json" \
  -d '{"brandId":"<brandId>","content":{"body":"Hello from our agent"},"targets":[{"channelId":"<channelId>"}]}'
```

If valid, send the same body to `POST /v1/posts` with `Idempotency-Key`, plus:
- `"schedule": {"at": "2026-11-02T09:30:00+01:00", "timezone": "Europe/Madrid"}` (omit or null = draft)
- `"submitForApproval": true` so a person sees it
- `"memory": {"topic": "...", "mainClaim": "..."}` so the brand's content memory stays useful

Post shape: `content.body`, optional `content.thread` (list of `{body, delayMinutes}`), optional `content.firstComment`, optional `content.mediaIds`. Per channel: `targets[].override` with its own `body`.

**5. A week at once (recommended for autopilot)**

```
POST /v1/brands/{brandId}/batches
{"posts":[{"channels":["<channelId>"],"text":"...","scheduledAt":"auto","topic":"...","mainClaim":"..."}]}
```
`"scheduledAt": "auto"` spreads posts over the week within the brand's daily caps. A person releases the batch with one approval. `GET /v1/brands/{brandId}/autopilot/prompt` returns a ready-made weekly instruction.

**6. Media**

`POST /v1/media/uploads` gives a presigned URL. PUT the file there, then `POST /v1/media/{mediaId}/complete`, then use the `mediaId` in posts.

**7. Follow up**

- `GET /v1/posts?brandId=...&status=...`: list posts
- `GET /v1/posts/{postId}`: per-channel delivery state
- `GET /v1/posts/{postId}/timeline`: every attempt with platform response codes
- `PATCH /v1/posts/{postId}`, `POST /v1/posts/{postId}/reschedule` (`{"schedule": null}` to unschedule), `DELETE /v1/posts/{postId}`

## Rules

- You cannot approve. `POST /v1/approvals/{id}/decision` and batch approve are humans only. Tell the user where to approve: the **Approvals** page in the app.
- If something looks wrong (bad post live, wrong account), call `POST /v1/brands/{brandId}/autopilot/stop` at once, then tell the user. `.../resume` restarts.
- Don't invent stats, quotes or features. Use product-context only.
- Report back what you scheduled: channel, time (in the user's timezone), and status.
