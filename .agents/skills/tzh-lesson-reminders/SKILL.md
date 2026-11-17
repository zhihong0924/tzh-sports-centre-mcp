---
name: tzh-lesson-reminders
description: Find today's eligible lesson enrollments, prepare saved website-template WhatsApp messages, send through an owner-approved WhatsApp session and record observed sends. Use for class attendance reminders, not lesson fees or stringing.
---

# Today's class reminders

Requires the matching deployed three-tool contract on `tzh_sports_centre`, an
active administrator and `lessons:manage`. This skill is prepared for an upcoming
release: confirm discovery before use. Missing tools/permissions are reported;
never fall back to SQL, Prisma, source, shell or internal HTTP. No Cloud API calls.
Token access alone does not authorize messages. Respect the owner's explicit or
standing approved recipients, language, template and schedule scope; do not
activate/change schedules or templates without a request.

1. Call `list_lesson_reminders` with an optional limit (default 20, maximum 50).
   Today is server-resolved in `Asia/Kuala_Lumpur`. Preserve enrollment IDs and
   opaque cursors. Skip every non-null `skipReason`. Scan up to five pages, send
   at most the approved cap (maximum 20), and report deferred/partial coverage.
   Process all eligible unsent enrollments for today in this run within the
   approved scope/cap, even when class is hours away. Do not invent per-class due
   times or a lead-time window, or wait until a class is nearly starting. Multiple
   classes for one student require separate reminders per enrollment; do not
   deduplicate by student or phone. Started classes remain skipped.
2. Immediately before each approved send, call `prepare_lesson_reminder` with its
   enrollment ID and approved `language` (`en`, `zh`, `ms`; default `en`). It reads
   the latest saved website wording, recipient, venue and eligibility. Never
   infer language, rewrite wording or substitute billing/guardian contacts.
   Disabled/out-of-hours/started/cancelled/absent/already-sent/invalid-contact
   records must not be sent. A stale cursor requires a new scan.
3. Pass the returned `whatsappUrl` unchanged to the authorized connected browser.
   Necessary private MCP results and browser/computer tool arguments are permitted;
   separate runtimes need no shared-variable bridge. Keep phone/message/link out
   of summaries, screenshots and extra logs. Do not use shell/temp files or
   external services to transfer private content.
4. Follow **Open app → Open WhatsApp**, or approved **Continue to WhatsApp Web**.
   Verify recipient and exact prefilled text before Send. Stop on sign-in,
   permission, handoff or mismatch failures. Do not search numbers or manually
   type/paste as an automatic fallback. A manual alternative requires an explicit
   owner choice and never bypasses denied permission.
5. Only after observing the exact new outgoing message in its verified chat, call
   `mark_lesson_reminder_sent({ enrollmentId, confirm:true })`. A Send click,
   chat-list preview or empty composer is insufficient. One reminder is recorded
   per enrollment; changing wording/language never authorizes another. Recording
   may finish after contact hours; it is not a delivery/read receipt.
6. Inspect an uncertain send before retrying. Do not click Send again merely
   because verification failed. If a send was observed but recording failed,
   retry only its same enrollment/confirmation. Recording retries preserve the
   first actor/time. Without an observed send leave the sent flag unchanged.

Use one sender at a time across reminder schedules. Overlap or a lost send
acknowledgement can duplicate messages; there is no reservation or exactly-once
promise. Report sent/skipped/failed/deferred counts and minimal private identifiers
only when needed. Do not change attendance, payments or enrollments. Website
editing is **Admin → WhatsApp Reminders → Class reminders** at `/admin/reminders`;
**Lesson fees** is the independent CSV tool. Supervised setup remains in `SETUP.md`.
