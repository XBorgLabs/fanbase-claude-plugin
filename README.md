# FanBase Copilot for Claude

The FanBase Copilot plugin for Claude. It connects Claude to the FanBase MCP server so Claude can
work your community inbox: see what needs a human across Discord, Instagram, X, WhatsApp and email,
read a fan's history, and reply in your brand voice — without opening the dashboard. It works in
claude.ai, the desktop and mobile apps, Cowork, and Claude Code.

## Install and connect

1. Add **FanBase Copilot** from the directory under **Customize > Plugins**.
2. Open the plugin's **Connectors** tab and select **Connect** on `fanbase`. On Team and Enterprise
   plans an Owner adds the connector for the organization first, and each member then connects.
3. The browser opens the FanBase consent screen. Sign in with the one-time code sent to your email,
   or **Continue with Google**.
4. Choose the **teamspace** Claude should operate on, and confirm the permissions.
5. Return to Claude and confirm the FanBase tools are listed.

In Claude Code the plugin connects to the server directly; run `/mcp` and authenticate `fanbase`.

The plugin points at the production endpoint `https://api.copilot.fanbase.gg/mcp` and discovers OAuth
from the server. There is no API key to paste, and nothing secret belongs in `.mcp.json`.

**Posting permission.** *MCP can Post to Socials* is granted on that consent screen and is on by
default. It gates every outbound tool — replies, publishing, scheduling changes, automations. There
is no setting elsewhere in the app to change it afterwards: run the connection flow again.

**Teamspace access is re-checked on every call.** Losing membership takes effect immediately, not at
the next session.

## Try it

With the connector connected, ask Claude:

```text
Use FanBase to show me what needs a reply today, grouped by bucket.
```

Claude should call `list_activity` with no arguments and come back with open items, the per-bucket
counts and the totals. A successful call confirms both the connection and your teamspace selection.

Then follow it with:

```text
Read that Instagram thread in full, pull up the fan's history, and draft a reply in our brand voice.
Show me the draft — don't send it.
```

That exercises `list_activity` with a `threadId`, `get_fan_activity` and `get_brand_voice`, and stops
short of `reply_to_activity`. Sending is a separate, explicit step.

## What's included

| Component | What it provides |
| --- | --- |
| MCP connector | The production FanBase MCP server (27 tools) over streamable HTTP with OAuth 2.1 |
| `triage-community-inbox` | The inbox loop: read the feed, read the thread, load the voice, reply or dismiss |
| `look-up-a-fan` | Find someone in the CRM, read their cross-platform history, rank the community |
| `publish-in-brand-voice` | Draft against the brand voice, publish or schedule, manage what is queued |

The three skills cover the core workflows. The rest of the surface — analytics, automations,
sentiment, knowledge — is available to Claude as soon as the connector is connected, just without a
skill guiding it. On claude.ai and the desktop app, some tools also render interactive cards in the
conversation: platform connections, media uploads and the brand voice.

## Tools

| Group | Tools |
| --- | --- |
| Activity feed | `list_activity`, `reply_to_activity`, `update_activity_status` |
| Community & fans | `list_crm`, `get_fan_activity`, `query_fan_events`, `mark_fans_internal` |
| Analytics | `get_account_analytics`, `get_analytics_trend` |
| Content | `post_content`, `draft_post`, `list_posts`, `update_scheduled_post`, `request_media_upload` |
| Brand voice | `get_brand_voice`, `update_brand_voice`, `generate_brand_voice` |
| Sentiment | `analyze_sentiment` |
| Jobs | `check_job` |
| Knowledge | `search_documents` |
| Automations | `list_automations`, `get_automation_analytics`, `create_automation`, `update_automation` |
| Connections | `list_platform_connections`, `list_discord_channels`, `lookup_socials` |

`draft_post`, `analyze_sentiment`, `generate_brand_voice` and `request_media_upload` are
asynchronous: they return a `jobId` that `check_job` polls.

## What this can do to your accounts

Most of the surface is read-only. Seven tools act outside the conversation, and they reach real
people: `reply_to_activity` sends a DM, email, comment or mention reply; `post_content` and
`update_scheduled_post` publish, reschedule or cancel public posts; `create_automation` and
`update_automation` change auto-replies that send on their own; `update_brand_voice` and
`mark_fans_internal` change what the whole team sees.

Three things stand between Claude and those:

- **The posting grant.** Publishing, replying and scheduling changes are refused unless the
  organization granted *MCP can Post to Socials* on the consent screen.
- **Claude's tool permissions.** Every tool is annotated. Read-only tools can run without a
  confirmation; `update_scheduled_post` and `update_brand_voice`, which can cancel a post or overwrite
  the voice, always ask; the rest follow your connector permission settings.
- **The skills.** They tell Claude to show you the exact text and destination, and get an explicit
  yes, before sending, publishing, cancelling or marking fans internal. Drafting or reviewing is never
  approval to send.

Automations created through the connector always start paused.

## What data it sends

Tool results — messages, fan records, analytics — come back into the conversation and into Claude's
context, the same as any other connector. Only what a tool returns, and only for the teamspace you
picked. The plugin itself stores nothing and makes no network calls of its own; everything goes
through `api.copilot.fanbase.gg`.

FanBase records every call server-side: the organization, the user, which OAuth client made it, the
tool name, a sanitized copy of the arguments with secret-looking keys redacted and long values
truncated, and whether it succeeded. That record is how a teamspace audits what its agents did.

The plugin holds no credentials. Authentication is OAuth, the token is held by Claude, and
`.mcp.json` carries nothing but a URL.

## Local development

```bash
claude plugin validate ./fanbase-copilot
claude --plugin-dir ./fanbase-copilot
```

The first checks the manifest, `.mcp.json` and every skill's frontmatter. The second starts Claude
Code with the plugin loaded from your working copy, so the skills appear as
`/fanbase-copilot:triage-community-inbox` and friends.

## Troubleshooting

| Symptom | Action |
| --- | --- |
| No FanBase tools in the conversation | Open the plugin's **Connectors** tab and **Connect** `fanbase` |
| `401` on every call | The OAuth token expired or access was revoked. Disconnect and connect again |
| A posting tool is refused | The teamspace has not granted *MCP can Post to Socials*. Re-run the consent flow |
| Feed is empty or stale | Ask Claude to run `list_platform_connections` — a platform is likely disconnected |
| Wrong teamspace's data | Reconnect and pick the other teamspace at the consent step |

## Staging

To point a test install at staging, change the URL in `.mcp.json` to
`https://api.staging-copilot.fanbase.gg/mcp`. Do not ship that change — the directory listing must
point at production.

## Privacy and documentation

- Privacy policy: https://copilot.fanbase.gg/privacy-policies
- Terms of service: https://copilot.fanbase.gg/terms-of-services
- Documentation: https://copilot.fanbase.gg/docs
