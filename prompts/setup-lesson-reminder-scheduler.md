# Set up daily lesson reminders

Copy the prompt below into Codex in your updated TZH MCP workspace after the
matching website release. It asks for your choices before activating a schedule.

```text
Use $tzh-lesson-reminders to help me create a daily WhatsApp lesson-reminder
scheduler on this computer, using the production tzh_sports_centre connection.

Before creating it, ask me for the daily run time in Asia/Kuala_Lumpur, message
language (English/Chinese/Malay), approved WhatsApp Web or desktop session,
allowed contact hours and per-run message cap (maximum 20). Confirm that the
scope is all eligible students/guests with lessons today. Do not invent timing.
Check existing automations so we update a matching schedule instead of duplicating
one, and coordinate with any stringing sender so runs do not overlap.

Verify the three lesson-reminder MCP tools are available. Use read-only discovery
to check server enabled/contact-hours readiness. If unavailable or disabled,
report what TZH must configure; do not bypass it or activate a sending schedule.
Confirm supervised WhatsApp setup has passed, or arrange an explicitly approved
test with an owner-controlled recipient first. Do not create lessons or test on
other students. Show the final schedule/settings for my approval, then use the
Codex Automations tools to create or update it. My approval of those settings is
standing authorization for the matching scheduled sends and sent recording.

Include these instructions in the saved automation:
Use $tzh-lesson-reminders. Each run lists today's lessons in Asia/Kuala_Lumpur
and processes all eligible unsent enrollments within our approved scope and cap
immediately, even if class is hours away. Do not wait for a per-class due time,
15-minute window or class start. Separate classes for one student require separate
messages; do not deduplicate by student or phone. Skip every non-null skipReason,
including started, cancelled, absent and already-sent lessons. Page discovery,
report deferred coverage and prepare each enrollment freshly using our selected
language and the latest saved website template. Pass its WhatsApp URL unchanged
to our approved browser/app session, verify recipient and exact prefilled text,
and send once. Only after observing the exact new outgoing message in the
verified chat, mark that enrollment sent with confirm:true. Inspect uncertain
sends before retrying; never resend blindly. Stop and report sign-in, permission,
handoff or recipient/draft failures. Keep one sender at a time. Report only
sent/skipped/failed/deferred counts, never private contact/message/link details.
Do not edit templates, attendance, lessons, payments, tokens or server settings.

After setup, tell me the next run time. Explain that this computer, Codex and the
approved signed-in WhatsApp session must be available for the local schedule.
```
