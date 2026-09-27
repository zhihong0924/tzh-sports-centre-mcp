---
name: tzh-lesson-management
description: Use the tzh_sports_centre MCP tools to find concrete lessons and atomically preview or commit duration and per-student price changes. Trigger for changing one or more specific lesson durations, end times implied by duration, or per-student prices. Do not use for series-wide edits, lesson creation/cancellation, capacity, enrollment, monthly-flat customization, or payment settlement.
---

# TZH Lesson Management

Use only the `tzh_sports_centre` MCP connection. Do not fall back to shell,
database, source-code, or direct HTTP access. This workflow changes concrete
lesson occurrences only; a recurring lesson's parent rule and siblings remain
unchanged.

## Workflow

1. Call `find_manageable_lessons` with a bounded date range and practical
   filters. Use only returned stable `lessonId` values. Names, chat claims, and
   list positions are never authoritative. If several lessons remain
   plausible, show their date, time, court, lesson type, duration, and RM price
   and ask the administrator to choose.
2. Build one full batch of distinct concrete lesson IDs. Each item must propose
   duration, per-student price, or both. Omitted fields preserve the current
   snapshot. Durations are positive 30-minute increments; prices are
   non-negative integer cents when sent to the tool, but must be presented to
   the administrator as RM.
3. When using a duration not listed as active for that lesson type, include
   `confirmCustomDurationFee: true` only after the administrator has confirmed
   that the displayed fee should be retained or has supplied a new price.
   Monthly-flat customization is unsupported.
4. Call `preview_lesson_management_batch`. Preview writes nothing. If any item
   is missing, duplicate, invalid, ineligible, stale, outside operating hours,
   or conflicts with a booking, recurring booking, lesson, training group, or
   another lesson in the same batch, report the reason and state clearly that
   no lesson changed.
5. Show the complete successful preview: every lesson's stable ID, lesson type,
   date, court, recurring-occurrence context, current and proposed duration,
   end time, RM price, customization state, all downstream effects, and expiry.
   State that no change has occurred.
6. Ask whether the administrator literally approves that exact full batch. A
   request to continue, finish, or take the next step is not approval.
7. Generate one stable idempotency key for the logical batch. Only after
   explicit approval, call `commit_lesson_management_batch` with the exact
   opaque `previewToken`, that idempotency key, and `confirm: true`. Never edit,
   decode, reconstruct, or substitute the preview token.

## Results and failures

- Report whether the outcome was discovery, previewed, committed, or replayed.
  Discovery and preview never write lesson data.
- Commit is atomic: every lesson changes or none do. A rejected, stale,
  conflicting, expired, or concurrently invalidated batch changes no lessons.
- An identical uncertain retry must reuse the exact preview token and
  idempotency key. `replayed: true` means the original operation was returned
  and no duplicate changes were made. Never reuse the key for changed content.
- If a preview expires or becomes stale, preview the complete current batch
  again, show it in full, and request new approval.
- If authentication lacks `lessons:manage`, ask TZH for a lesson-enabled
  replacement token through a secure channel. Never ask the user to paste a
  bearer token into chat.
- Never claim that a successful occurrence edit changed a recurring parent
  rule, sibling occurrence, existing invoice, enrollment, or payment.
