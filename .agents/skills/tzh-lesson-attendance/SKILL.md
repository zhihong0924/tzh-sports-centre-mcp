---
name: tzh-lesson-attendance
description: Use the tzh_sports_centre MCP tool to query registered-student lesson attendance, missing attendance, recorded outcomes, or due state over a Malaysia-time preset or bounded custom date range. Trigger for attendance review, completed lessons awaiting attendance, attendance history, or filtering attendance by lesson, court, lesson type, student, or stored status. This skill is read-only and never records or changes attendance.
---

# TZH Lesson Attendance Query

Use only the `tzh_sports_centre` MCP connection. Do not fall back to shell,
database, source-code, or direct HTTP access. This workflow is strictly
read-only: it cannot record attendance, infer an outcome, send a reminder, or
change a lesson, enrollment, replacement, fee, or notification.

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

## Failures

- If authentication lacks `attendance:read`, ask TZH for an
  attendance-read-enabled replacement token through a secure channel. Audit,
  points, and lesson-management permissions do not imply attendance access.
- Report malformed, mixed, inverted, oversized, contradictory, or stale-cursor
  errors faithfully and correct the query rather than guessing results.
- Never ask the user to paste a bearer token into chat.
