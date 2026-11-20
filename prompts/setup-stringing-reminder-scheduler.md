# Set up stringing reminders

Copy the prompt below into Codex in your updated TZH MCP workspace. It asks for
your timing and stage choices before activating a schedule.

```text
Use $tzh-stringing-reminders to help me create a WhatsApp stringing-reminder
scheduler on this computer, using the production tzh_sports_centre connection.

Reuse choices and successful supervised setup already confirmed for this approved
WhatsApp session. Ask only for missing or changed facts, grouped in one question.
Before creating it, confirm which stages to include: first payment reminder,
second payment reminder, ready-for-pickup, feedback and due restring reminders.
Confirm run days/time in Asia/Kuala_Lumpur, the approved WhatsApp Web or desktop
session and an optional per-run message cap (any positive whole number or
explicitly no cap). The approved schedule controls sending time; do not ask for
separate server contact hours or impose a fixed maximum of 20. For each
selected payment stage, ask for its whole-day waiting period: first from received
date, second from first recorded send. Do not guess waiting days. Explain that
feedback is once, three days after collection; restring follows the job's saved
due date. Use saved website templates and customer message languages.

Check existing automations so we update a matching schedule instead of duplicating
one. Coordinate with any lesson sender so runs do not overlap. Verify all four
stringing-reminder MCP tools through read-only discovery. There are no server
enable/contact-hour settings for stringing; pausing this scheduler stops future
runs (stop an already-running sender separately). If tool discovery or
authentication is blocked, report what TZH must configure; do not bypass it
or activate a sending schedule. Confirm supervised WhatsApp setup has passed,
or arrange an explicitly approved owner-controlled test first. Do not create
jobs or test on other customers. Show final stages, waiting periods and schedule
for my approval, then use Codex Automations tools to create or update it. My
approval is standing authorization for matching scheduled sends and recording.

Include these instructions in the saved automation:
Use $tzh-stringing-reminders with our selected stages, schedule and waiting days.
Supply the approved waitingDays in both list and fresh prepare calls for each
payment stage; omit it for other stages. Respect sent flags, eligibility and our
approved schedule. Page discovery until complete or our optional cap/runtime is
reached; apply our cap only if selected, and one job per customer per run. Report
deferred coverage. There is no fixed maximum of 20. Immediately before each send,
prepare the exact job/stage using its saved website wording and customer language.
Pass the returned WhatsApp URL unchanged to our approved browser/app session,
verify recipient and exact prefilled text, then send once. Only after observing
the exact new outgoing message in that verified chat, mark that job/stage sent
with confirm:true. For an eligible missing feedback request, follow the skill's
explicit creation and stable retry-key instructions; never replace revoked links.
Inspect uncertain sends before retrying; never resend blindly. Stop and report
sign-in, permission, handoff or recipient/draft failures. Use one sender at a time.
Report only sent/skipped/failed/deferred counts, never private contact/message/link
details. Do not change templates, customer language, payments, job progress,
tokens, server settings or unapproved reminder stages.

After setup, tell me the next run time. Explain that this computer, Codex and the
approved signed-in WhatsApp session must be available for the local schedule.
```
