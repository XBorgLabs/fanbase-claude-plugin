---
name: look-up-a-fan
description: Find someone in the FanBase CRM, read their cross-platform history, and answer questions about who the community's most engaged people are. Use when the user asks about a specific fan, wants a leaderboard or a breakdown of engagement, or wants staff and test accounts excluded from reports.
---

# Look up a fan

`list_crm` finds people. With no arguments it returns the most engaged first; `search` fuzzy-matches
username and display name. Filter with `platforms` (every listed platform must be present, so two
entries find cross-platform fans), `minEngagement`, `activeAfter`, or `minPlatforms`. Each result
carries the `clusterId` everything else keys off, and a stats map — read the keys from there rather
than guessing them, because balance keys are named by the community's own resource slug.

`get_fan_activity` takes a `clusterId` and returns that one person's timeline across every platform,
newest first. `query_fan_events` is the community-wide version: without `groupBy` it answers "what
happened", with it "how many", and `groupBy: "fan"` over a date window is a leaderboard.

Rank over a time window with `query_fan_events`, not with `list_crm`. The `statKey` filters on
list_crm are all-time totals, so they answer a different question than "who was most active this
month" and will quietly disagree with it.

Fans marked internal are already excluded from every list, timeline and count. `mark_fans_internal`
adds to that set, and it is a real write: it takes effect immediately everywhere, and unmarking is
only possible from the web app. Name the people first and get a yes.

Do not invent a `clusterId`, a stat key or an activity `type`. Each comes from a previous call or
from the tool's own description. If a fan cannot be found, say so rather than reporting on a
near-match.
