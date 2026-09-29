---
name: tzh-lesson-attendance
description: Use the tzh_sports_centre MCP tools to query lesson attendance and, only with explicit approval, preview and atomically record stable-ID attendance updates. Trigger for attendance review, missing or recorded outcomes, marking students present or absent, mark-all-present, or centre cancellation. Querying is read-only; recording follows a signed preview and confirmed commit workflow.
---

# TZH Lesson Attendance

Use only the `tzh_sports_centre` MCP connection. Do not fall back to shell,
database, source-code, or direct HTTP access. The query workflow is strictly
read-only. Querying and recording require `lessons:manage`; recording also
follows the staged preview/approval/commit workflow below. Existing tokens
with `lessons:manage` can use these tools after server deployment. Never infer
an outcome, send reminders, or bypass the tools.

## Query workflow

1. Call `query_lesson_attendance` with exactly one range form:
   - a preset: `today`, `yesterday`, `this_week`, `last_week`, `this_month`, or
     `last_month`; or
   - an inclusive `fromDate` and `toDate` in `YYYY-MM-DD` format, covering no
     more than 366 days.
   The server resolves presets, current dates, Monday week starts, and month
   boundaries in `Asia/Kuala_Lumpur`.
2. Choose `attendanceState` deliberately: `missing` for completed scheduled
   lessons whose registered enrollment is still attendance-pending and has no
   explicit outcome; `recorded` for stored outcomes; or `all` to include
   recorded, missing, not-yet-due, and explicitly included guest rows.
3. Optionally filter by stored attendance status or stable `lessonId`, numeric
   `courtId`, `lessonTypeId`, or `studentId`. Stored-status filtering cannot be
   combined with `attendanceState: missing`. Use only stable IDs returned by
   TZH tools or supplied by an authoritative workflow; never target a person
   from a name, chat claim, or list position.
4. Guests are excluded by default because TZH stores no `StudentAttendance`
   record for them. Set `includeGuests: true` only when needed; report them as
   `not_tracked`, never missing. Mention `excludedGuestCount` when non-zero.
5. Present the resolved range, Malaysia timezone, `asOf` timestamp, whole-query
   lesson/student/state/status summary, and returned rows. Each row may include
   only the stable IDs, display name, lesson schedule/type/court/status,
   enrollment status, derived attendance state, stored outcome/time, and
   `allowedActions` returned by the server. Do not add or request contact,
   payment, receipt, absence-reason, credential, or unrelated account data.
6. When `hasMore` is true, call the same query with every filter unchanged and
   pass `nextCursor` unchanged. Do not decode, edit, reconstruct, or reuse a
   cursor with different filters.

## Meaning and reporting

- A future lesson, including one later today, is `not_due`, not missing.
- Preserve stored statuses such as `present`, `eligible_absence`,
  `late_absence`, `no_show`, and `centre_cancelled`; billing or enrollment
  status is never proof of attendance.
- A replacement-funded enrollment cannot earn another replacement. Report only
  server-returned `allowedActions`; do not infer additional actions.
- Summary counts describe the complete filtered query, not only the page.
  `returnedRowCount` describes the current page.
- State which range and filters were queried and that no data changed.

## Recording workflow

1. Query first and identify every target by returned stable `lessonId` and
   `enrollmentId`. Never target by name, email, list position, or chat claim.
2. Build one bounded complete batch for the administrator's request:
   - per student: `present`, `eligible_absence`, or
     `absent_no_replacement`, with an optional reason;
   - per lesson: `mark_all_present` or `centre_cancelled`.
   Do not mix a lesson-wide action with per-student actions for that lesson.
3. Call `preview_lesson_attendance_updates`. This writes nothing. Show every
   target's current enrollment/attendance state, proposed stored outcome,
   reason, affected enrollment IDs, replacement issuance/restoration, seat or
   slot release, lesson cancellation, unchanged billing/invoice treatment,
   idempotent no-op, and preview expiry.
4. Ask separately whether the administrator approves that exact full preview.
   “Continue”, “finish”, silence, or a generic next-step response is not
   approval. Never decode, edit, or reconstruct the opaque preview token.
5. Only after literal approval call `commit_lesson_attendance_updates` with the
   exact token, `confirm: true`, and a stable idempotency key. Reuse that key
   only for an uncertain retry of unchanged content.
6. Report the durable operation ID, affected lesson/enrollment IDs,
   action/outcome counts, replacements issued or restored, cancelled lessons,
   and whether the result was replayed. Never add contact, payment, receipt, or
   credential data.

The server rejects guests, premature non-cancellation attendance, duplicate or
ambiguous targets, replacement-funded eligible absences, stale/tampered/expired
previews, and conflicting idempotency-key reuse. A rejected preview or commit
changes nothing. `present` and absence outcomes preserve the existing charge;
`centre_cancelled` preserves canonical billable treatment, releases slots, and
issues or restores only the applicable replacement entitlement.

## Failures

- If authentication lacks `lessons:manage`, ask TZH for a token with the
  website's **Lesson management** control through a secure channel. The single
  scope authorizes both attendance query and recording; each recording still
  requires an exact preview and explicit approval.
- After an uncertain commit response, retry only the exact opaque preview token
  and the same idempotency key. If the preview is stale or expired, preview the
  full current batch again and obtain new explicit approval.
- Report malformed, mixed, inverted, oversized, contradictory, or stale-cursor
  errors faithfully and correct the query rather than guessing results.
- Never ask the user to paste a bearer token into chat.
