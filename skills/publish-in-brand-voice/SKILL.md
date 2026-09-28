---
name: publish-in-brand-voice
description: Draft, schedule and publish social posts through FanBase in the organization's brand voice, and manage what is already queued. Use when the user asks for a post drafted or published, asks what is scheduled, or wants a scheduled post edited, moved or cancelled.
---

# Publish in the brand voice

Load `get_brand_voice` before writing anything. It carries the identity, tone, topics, audience and
the do's and don'ts the copy has to hold to.

Prefer `draft_post` over writing the copy yourself when the user wants something on-brand: it drafts
against the voice and against what the community is actually saying. It needs a `platform`, and polls
are X and Discord only. It returns a `jobId` and saves nothing — poll `check_job` about every ten
seconds, and expect roughly a minute.

Show the draft and wait. `post_content` is what publishes, and it goes out to real audiences the
moment it is called. Give it one post object per platform with the copy tailored to that platform,
and set a future `scheduledAt` on any post that should be queued instead. Each post succeeds or
fails on its own and is reported separately, but invalid input rejects the whole call before anything
sends — fix it and resend the full batch. Published posts come back with a `url`; always give those
links to the user.

For media, call `request_media_upload` and give the user the link it returns. That job stays pending
until they actually upload, so keep polling `check_job` rather than assuming it failed, then put the
returned `publicUrl` into that post's `mediaUrls`.

`list_posts` shows what is scheduled, published or failed — a failed item carries `publishError`.
`update_scheduled_post` edits, reschedules or cancels something that has not gone out; load it with
`list_posts` first, since `edit_content` merges over the current content. Cancelling is permanent and
a published post cannot be changed, so confirm before either.

Publishing, scheduling changes and replies all need the organization's MCP posting permission,
granted on the consent screen. If it was not granted, the call is refused and the fix is to run the
connection flow again — there is no setting elsewhere in the app.
