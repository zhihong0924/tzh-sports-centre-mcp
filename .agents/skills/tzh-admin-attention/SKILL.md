---
name: tzh-admin-attention
description: Query current administrator work due today, tomorrow, this week, or a bounded Malaysia-calendar range through the read-only tzh_sports_centre MCP tool. Use for scheduled admin task digests and questions about overdue or undated admin work; do not send reminders or change business state.
---

# TZH Admin Attention

Use only the configured `tzh_sports_centre` MCP connection. Never inspect the
private database, application source, internal HTTP APIs, or shell commands as
a fallback for TZH data. A connected personal token must have the independent
`admin-attention:read` permission and belong to an active administrator. Do not
ask anyone to paste a token into chat.

## Query workflow

1. Map ordinary phrases to strict inputs for `query_admin_attention`:
   - “tasks due today” → `{ "preset": "today" }`;
   - “what needs attention tomorrow” → `{ "preset": "tomorrow" }`;
   - “this week's admin work” → `{ "preset": "this_week" }`.
   Use `fromDate` and `toDate` in `YYYY-MM-DD` only for an explicitly requested
   inclusive custom range of no more than 31 days. Do not pass a natural
   language question to the tool or mix preset and custom fields.
2. State the returned `asOf`, `Asia/Kuala_Lumpur` range, and Monday–Sunday week
   interpretation. Show `overdue`, `due_in_window`, and `undated_backlog`
   separately. Include complete per-section counts, `totalActionable`, and
   `outsideWindowDated`, even if the current page or due section is empty.
3. Summarize `providersQueried` and `explicitExclusions`. A zero due-today count
   never means all admin work is complete. Neither a creation timestamp nor a
   preferred trial date becomes an invented due date.
4. Return only the stable queue/feature ID, entity ID, action label, current
   state, real due date and provenance when present, and the returned website
   link needed for triage. Do not request or expose contact details, receipt
   or proof contents or URLs, medical notes, payment secrets, or unrelated
   student history.
5. For another page, pass `nextCursor` unchanged with the same range and
   chosen page size. Counts cover the whole live query, not only the page.
   A completed action disappears on a fresh query. If a cursor is stale,
   restart from the first page; do not decode or edit the cursor.

The tool is read-only. It does not send messages, mark items read, create a
scheduled chat, record attendance, approve a receipt, or change any business
state. Use each returned deep link to complete an action in its existing
website workflow. Do not infer that a scheduled booking, unpaid member charge,
webhook event, or historical record is an admin task when it was excluded by
the server's coverage decision.

## Failures

A provider failure invalidates the entire result. Report it and do not claim
an empty queue. Retry a failed read once with the same input; report continued
failure. A missing `admin-attention:read` permission requires a new or updated
token from TZH through a secure channel; lesson-management, points, and audit
permissions do not grant this read scope.
