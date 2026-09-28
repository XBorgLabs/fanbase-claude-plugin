---
name: triage-community-inbox
description: Work the FanBase activity feed — the DMs, emails, comments and mentions that need a human across every connected platform — and reply or dismiss. Use when the user asks what needs a reply, wants the inbox cleared, or asks about a conversation.
---

# Triage the community inbox

Call `list_activity` with no arguments first. That answers "what needs me?", and the same page
returns the organization's classification buckets with counts and the overall totals. Bucket keys
are configured per organization: only pass a `bucket` that came back in that block. `unclassified`
and `all` are the two you never have to look up.

Read before answering. Pass an item's `threadId` back to `list_activity` for the whole conversation,
and its `fanClusterId` to `get_fan_activity` for who you are talking to. `search` matches the message
and the author, so it finds a person by handle or everything said about a topic. For facts about the
product, pricing or policies, check `search_documents` rather than answering from memory.

Load `get_brand_voice` before drafting anything. Write against it rather than from a general sense of
the brand.

`reply_to_activity` sends through whichever platform the item arrived on and marks it handled. It
reaches a real person immediately, so show the user the exact wording and get a yes first. It needs
the organization's MCP posting permission, granted on the consent screen; without it the call is
refused and re-running the connection flow is the only way to change that.

`update_activity_status` is the no-send path: `ignored` clears spam and anything needing no answer,
`open` reopens it. There is no `handled` here — replying is what sets that. Name the items before a
bulk sweep, because it changes what the whole team sees.

Never invent an `activityId`, `threadId`, `fanClusterId` or bucket key; each comes from a previous
call. If the feed is empty or stale, run `list_platform_connections` — that is usually a broken
connection rather than a quiet week.
