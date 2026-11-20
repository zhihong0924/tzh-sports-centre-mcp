# Set up the TZH Sports Centre MCP workspace

Keep this Git repository intact. It contains:

```text
tzh-sports-centre-mcp/
├── AGENTS.md
├── SETUP.md
└── .agents/
    └── skills/
        ├── tzh-student-account-audit/
        │   └── SKILL.md
        ├── tzh-point-assignment/
        │   └── SKILL.md
        ├── tzh-lesson-management/
        │   └── SKILL.md
        ├── tzh-lesson-reminders/
        │   └── SKILL.md
        ├── tzh-lesson-attendance/
        │   └── SKILL.md
        ├── tzh-admin-attention/
        │   └── SKILL.md
        ├── tzh-manual-invoice/
        │   └── SKILL.md
        └── tzh-non-member-invoice/
            └── SKILL.md
```

`AGENTS.md` supplies instructions for every task started from this workspace.
Codex discovers the skill from `.agents/skills`; it reads the full `SKILL.md`
only when the request matches a skill or you invoke
`$tzh-student-account-audit`, `$tzh-point-assignment`,
`$tzh-lesson-management`, `$tzh-lesson-attendance`, or
`$tzh-admin-attention` or `$tzh-manual-invoice` explicitly. A loose file
named `skills.md` is not the Codex skill format.

## 1. Store the bearer token on macOS

TZH supplies the token separately through a secure channel. A TZH administrator
normally creates a separately named, independently revocable token from the
website's **MCP Access Tokens** workspace; its plaintext is shown only once.
The token must include the permissions needed by the intended workflow. The
website offers **Student account**, **Member points**, **Lesson management**,
**Read admin attention**, and **Manual invoices** controls. Read admin attention grants only the
independent `admin-attention:read` permission. Lesson management grants
duration/price changes plus attendance reading and recording through the
single `lessons:manage` permission. Existing
tokens with that scope can use the deployed attendance tools.
Never save it in this folder, `AGENTS.md`, `SKILL.md`, chat, screenshots, or
source control.

In Terminal, capture it without displaying it:

```zsh
read -s "TZH_SPORTS_CENTRE_MCP_TOKEN?Paste the TZH token (hidden): "
echo
export TZH_SPORTS_CENTRE_MCP_TOKEN
launchctl setenv TZH_SPORTS_CENTRE_MCP_TOKEN "$TZH_SPORTS_CENTRE_MCP_TOKEN"
unset TZH_SPORTS_CENTRE_MCP_TOKEN
```

Fully quit Codex after changing the environment. The value may need to be set
again after logout or restart. To remove it later:

```zsh
launchctl unsetenv TZH_SPORTS_CENTRE_MCP_TOKEN
```

Do not run `launchctl getenv TZH_SPORTS_CENTRE_MCP_TOKEN` while screen sharing;
it prints the secret.

## 2. Configure the MCP connection

Add this block to `~/.codex/config.toml`, replacing only the domain placeholder:

```toml
[mcp_servers.tzh_sports_centre]
url = "https://YOUR-TZH-DOMAIN/api/mcp"
bearer_token_env_var = "TZH_SPORTS_CENTRE_MCP_TOKEN"
required = false
startup_timeout_sec = 10
tool_timeout_sec = 60
default_tools_approval_mode = "writes"
```

The server is optional so a temporary outage does not prevent Codex from
starting. Do not add an `enabled_tools` list: the shared endpoint can publish
new reviewed TZH tools without requiring another configuration edit. Server-side
authorization remains authoritative.

For a TZH-owner local test only, the URL may temporarily be:

```text
http://localhost:3000/api/mcp
```

A customer using the deployed service must use its public HTTPS URL.

## 3. Create the Codex project

1. Fully quit and reopen Codex.
2. Create a local Codex project or edit an existing project.
3. Add this entire `tzh-sports-centre-mcp` folder.
4. Set it as the project's main folder.
5. Start a new task from that project.

The main folder matters: Codex uses it as the default location for discovering
project `AGENTS.md`, `.agents/skills`, and project configuration.

## 4. Verify discovery

1. Enter `/mcp` and confirm `tzh_sports_centre` is enabled and authenticated.
2. Enter `/skills`, or type `$`, and confirm `tzh-student-account-audit`,
   `tzh-point-assignment`, `tzh-lesson-management`, and
   `tzh-lesson-attendance`, `tzh-admin-attention`, and `tzh-manual-invoice` appear.
3. Run this read-only smoke test with a safe search value:

```text
Use $tzh-student-account-audit and call search_student_accounts from
tzh_sports_centre with {"query":"EXAMPLE NAME OR EMAIL"}. Do not create or
modify anything.
```

The expected result is a bounded list of matching active students or an empty
list. If several students match, continue only after choosing by stable student
ID.

For a point-enabled token, run a second read-only smoke test:

```text
Use $tzh-point-assignment and call list_active_point_policies from
tzh_sports_centre. Do not preview or commit an assignment.
```

Point assignment uses `search_point_members`, `list_active_point_policies`,
`preview_points_assignment`, and `commit_points_assignment`. The preview writes
nothing and must be shown in full. Commit requires explicit approval and the
exact opaque preview token. This workflow awards positive points only.

For a lesson-management token, run a read-only discovery smoke test:

```text
Use $tzh-lesson-management and call find_manageable_lessons from
tzh_sports_centre for a small YYYY-MM-DD date range. Do not preview or commit
anything.
```

Lesson management uses `find_manageable_lessons`,
`preview_lesson_management_batch`, and `commit_lesson_management_batch`.
Discovery returns the active student count and minimal roster names with court,
time, duration, and RM price so the intended occurrence can be identified; it
does not expose contact, payment, or receipt details, and labels truncated
rosters.
Preview is read-only and must be shown in full. Commit requires literal
approval of that exact preview, `confirm: true`, and a stable idempotency key.
Any rejected batch changes no lessons.

For a token with **Lesson management** access, run this read-only attendance
smoke test:

```text
Use $tzh-lesson-attendance and call query_lesson_attendance from
tzh_sports_centre with {"preset":"today","attendanceState":"all"}. Do not
record or modify anything.
```

The result resolves the range in `Asia/Kuala_Lumpur`, reports whole-query
summary counts, and returns bounded per-enrollment rows. Guests are excluded by
default; when explicitly included they are `not_tracked`. A later lesson is
`not_due`, and no billing or enrollment state is treated as proof of attendance.
Use `nextCursor` unchanged with the same filters when `hasMore` is true.

For a token with **Read admin attention** access, run this read-only smoke test:

```text
Use $tzh-admin-attention and call query_admin_attention from
tzh_sports_centre with {"preset":"today"}. Report the resolved Malaysia range,
overdue, due-in-window, and undated sections, plus queried providers and
exclusions. Do not send messages or modify anything.
```

A customer's scheduled chat may call the same tool with `today`, `tomorrow`,
or `this_week`. This setup does not create or configure that chat. An empty
due-today section does not mean the overdue or undated backlog is empty.

For attendance recording, first use the read tool to identify stable
lesson/enrollment IDs, then call `preview_lesson_attendance_updates`. Show the
complete preview and make no changes until the administrator literally approves
that exact batch. Commit with `commit_lesson_attendance_updates`, the opaque
preview token unchanged, `confirm: true`, and a stable idempotency key. Exact
retries reuse the same token and key; stale or expired previews require a new
preview and approval.

The catalogue also includes `remove_student_audit_entry` for removing one entry
from a draft and `delete_student_audit_case` for permanently deleting a draft
or rejected case. Both require explicit confirmation and cannot reverse a
committed case.

For an existing audit, Codex uses `list_student_audit_cases` after confirming
the student. It resolves a unique case name/date match internally; if more than
one case is plausible, it asks you to choose from readable case details instead
of asking for an internal case ID.

During an audit, Codex should explain the completed stage and guide the next
choice—for example, after case creation it should ask which lesson, payment,
correction, or proof to add. It may continue draft work already requested with
complete inputs, but it must always show a successful review and request
separate approval for that exact validation version before commit. The review
includes the case and entry details, evidence-image counts, and the same
current/change/projected fee and replacement calculation shown by the web
workspace. It does not display or inspect the private proof images. All review
amounts are displayed in RM rather than cents.

Committed audit replacement obligations remain lifetime credits. The normal
live absence-token expiry does not apply; redemption is completed by an admin
through the website's replacement-booking workflow.

## 5. Troubleshooting and updates

- If the server is absent, check the exact URL, environment variable name, and
  `[mcp_servers.tzh_sports_centre]` table, then fully restart Codex.
- If the skill is absent, confirm the hidden `.agents` directory was included,
  the file is exactly `.agents/skills/tzh-student-account-audit/SKILL.md`, and
  this folder is the project's main folder. Start a new task after correcting it.
- Never paste a token into chat while troubleshooting. Ask TZH to revoke and
  replace any token that may have been exposed.
- If audit tools report a missing permission, ask TZH for a replacement token
  with `student-audit:manage`; do not attempt to use a points-only token.
- If `query_admin_attention` reports a missing permission, ask TZH for a
  separate token with **Read admin attention**. Audit, points, and lesson
  permissions do not imply `admin-attention:read`.
- If lesson or attendance tools report a missing permission, ask TZH for a
  replacement token with the website's **Lesson management** control. It grants
  `lessons:manage` for duration/price changes and attendance query/recording.
  Audit-only and points-only tokens do not authorize lesson tools. Existing
  `lessons:manage` tokens already have the needed scope.
- If an approved `commit_student_audit_case` call times out, report the timeout
  and retry only the exact same case ID and approved validation version. That
  retry is safe and does not repeat canonical records, invoices, or emails.
  Never substitute a newer validation version without a new review and explicit
  approval.
- To receive updated instructions or new skills, first confirm that
  `git status --short` is empty, then run `git pull --ff-only` from this
  repository's root. Do not discard unexpected local changes; ask TZH for help
  instead. After a successful pull, fully restart Codex and start a new task so
  it discovers the updated `AGENTS.md` and skills.


## Shared paid manual invoices

Use the `tzh-manual-invoice` skill for the shared manual-invoice workflow below.
The owner confirmed the then-existing functions on 2026-10-13 and the
invoice-idempotency migration on 2026-10-12. These dated checks do not establish
availability or migration readiness of later releases. Verify required tools
on the configured connection.

The website's **Manual invoices** token control grants
`invoices:manage` for registered-user and non-member already-paid invoices.
Existing invoice-enabled tokens cover both types; unrelated permissions do not
imply invoice authority. Owners should review recipients and revoke/reissue any
invoice token that should not gain member invoice access. Plaintext credentials
are delivered through the secure channel, never chat.

Verify discovery of `search_invoice_members`, `preview_manual_invoice` and
`commit_manual_invoice`. For an invoice-enabled token, a read-only smoke test is:

```text
Use $tzh-manual-invoice and call search_invoice_members from tzh_sports_centre
with a known test customer's name, email or phone fragment. Report the bounded
matches. Do not preview or create an invoice.
```

Select a registered account by its stable discovery ID, or explicitly select a
manually named non-member. The shared preview takes `customer` with
`type: "member"` and `userId`, or `type: "non_member"` and `name`, plus
Malaysia invoice date, positive RM line items, optional additional names,
`paymentAlreadyReceived: true` and a stable idempotency key. It automatically
places the primary customer name first and returns a complete read-only proposal.

Show all names, items, amounts, total, paid state, member ownership, points,
notification email, effects and expiry. Registered users earn RM1=1 point,
rounded down, and receive an eligible invoice email; non-members receive no
ownership, points or email. Additional names grant no account access. Commit
only after explicit approval of the exact signed preview, using its unchanged
token and `confirm: true`. Report invoice number, total RM, points and email
status. Email failure does not undo the committed invoice or points; a replay
never sends it again. The PDF is available in admin invoices and the registered
owner's My Invoices.

Exact approved-token retries are safe, including after expiry. Changed content,
stale member name/email/phone or an expired uncommitted token requires a fresh
preview and new approval. Never create another invoice to retry email or bypass
an uncertain result. If a required tool is missing, report a connection or permission problem and stop; never use private data through SQL/shell/internal HTTP.

The old non-member tool names and `tzh-non-member-invoice` skill remain for
compatibility. They cannot select a registered member. Prefer the shared tools
for new work.

## Off-site lessons — deployed 16 October 2026

The off-site migration and server release are deployed. Lesson discovery and
attendance may return `court.id = null` and an `Off-site: …` venue name. Treat
this as an external location with no centre court; never invent a court ID.
Numeric court filters exclude these lessons. Existing `lessons:manage` scopes
and exact preview/commit approval still apply. Off-site duration/price changes
reserve no centre slots and may fall outside centre operating hours. Lesson
creation remains on the website.


## Stringing reminders setup — scheduler setup required

Deploy the matching application, forward migrations and customer workspace before
using this in production. Existing active, unexpired tokens receive all six current permissions through
the owner-approved data-only release migration; secrets, owners, unrelated scopes
and expiry/revocation stay unchanged. Expired/revoked tokens are unchanged and
remain invalid. New tokens select **Stringing reminders**. Require a live active
administrator. Keep the stable signing secret and trusted HTTPS feedback origin.

Discover four tools: `list_stringing_reminders`, `prepare_stringing_reminder`,
`mark_stringing_reminder_sent`, `create_stringing_feedback_request`.
Refresh cached tool discovery after updating: the retired tool names are no longer
registered. Existing bearer secrets remain valid; the server recognizes the retired
permission internally and the data-only permission migration normalizes stored
scopes. Token expiry, revocation and other permissions are unchanged.
There are no policy, claim or recovery tools.
There is no customer pause checkbox or stored reminder policy.

The admin puts payment waiting periods in the scheduler prompt, for example:
“First payment reminder two days after receipt; second reminder four days after
the first recorded send.” Pass `waitingDays:2` for payment_first and `waitingDays:4`
for payment_second in every list/prepare call. Both accept 0–3650 whole Malaysia
calendar days. Missing values are rejected; never guess a default or save settings.
For ready/feedback/restring omit waitingDays. Disable a stage by excluding it from
the approved schedule. If timing changes, restart pagination with no cursor.
Feedback remains once, three days after collection; pickup requires Ready and
restring follows the job's pending due date. A revoked feedback link is never
replaced automatically; create a missing due request only when explicitly needed.

Agree on recipients, stages, saved EN/ZH/MS templates, the Malaysia-time schedule,
WhatsApp app and an optional per-run cap (any positive whole number or explicitly
no cap). Keep one job per customer per run; there is no fixed 20-message maximum. Run one sender
at a time; limits are client instructions, not server reservations. Sign in to the
approved WhatsApp desktop or Web session and enable connected computer/browser
access. Token access alone does not authorize sending. The client-approved
scheduler controls whether and when stringing reminders run. No server enable
or start/end-hour settings are required; retired STRINGING_REMINDERS and
STRINGING_OUTREACH enable/hour variables are ignored after deploying the updated
server. Pause the scheduler to prevent future runs; stop an already-running
sender separately. Refresh MCP discovery: enabled, contactHours, allowedNow and
runCap are no longer returned. Ordinary dry runs
only list candidates and never open WhatsApp or record sent status.

For a supervised test, use an owner-controlled recipient and eligible fictional
job. Prepare fresh, open the returned `whatsappUrl` unchanged in the connected
browser, and follow **Open app / Open WhatsApp** into the approved desktop app
(or **Continue to WhatsApp Web** for an approved Web session). Verify the intended
recipient and exact prefilled text before sending. Observe the exact new outgoing
message in that chat itself,
then call `mark_stringing_reminder_sent({ jobId, stage, confirm:true })`. Verify the
existing website sent status and, for restring, completed reminder. Do not expose
phones/messages/feedback links in logs, screenshots or summaries. No real customer
sends as development tests. Schedule activation remains a separate owner action.

If not sent or not observed, leave the sent flag unchanged. A later run can retry
eligible unsent work; the owner accepts duplicate risk after a send/recording
disconnect or overlapping runs. Check the chat when unsure; never invent Sent.
Stop on login, permission, recipient or configuration failures. MCP uses the same
website template formatter and Mark as sent writer. It does not change payment,
progress, fees, points, invoices or feedback responses.

Example scheduler prompt (fill and approve before activation):

```text
Use $tzh-stringing-reminders through tzh_sports_centre for OWNER_APPROVED recipients
and saved templates on APPROVED_MALAYSIA_SCHEDULE using APPROVED_WHATSAPP_APP.
For payment_first use waitingDays 2, measured from receipt; for payment_second use
waitingDays 4, measured from first recorded send. These are this schedule's admin
settings, not website settings. Skip any stage not approved here. Use the same
waitingDays for each stage's list and prepare calls. Omit it for ready, feedback
and restring. Run one sender and one job per customer per run. Apply only the
client-approved optional cap (a positive whole number or explicitly no cap); do
not impose a fixed maximum of 20. Page discovery in batches of up to 50 until
complete or the approved cap/runtime is reached; report deferred coverage. Respect existing sent flags and all returned skip reasons. Create a
missing due feedback request once if authorized. Prepare immediately before sending;
pass the prepared whatsappUrl directly as the authorized browser navigation
argument. Necessary MCP results and browser/computer tool arguments are allowed;
keep private fields out of reports, screenshots and unnecessary diagnostic logs.
Separate tool runtimes need no shared variables or private-transfer bridge. Do not
use shell/temp files or external services to transfer messages. Open the returned
whatsappUrl unchanged in the connected browser and follow the
Open app/Open WhatsApp handoff (or Continue to WhatsApp Web for approved Web use).
Verify the intended recipient and exact prefilled text. Do not search numbers,
rebuild URLs or type/paste messages as an automatic alternative. Stop and report
blocked handoff, sign-in or missing/mismatched drafts; only an explicit owner choice
permits a manual alternative. Never bypass denied permissions. Send once and observe
the exact new outgoing message in that chat itself, then mark that job/stage sent.
Chat-list previews and an empty composer alone are insufficient. Inspect uncertain
sends before retrying; do not click Send again just because verification failed. Leave status unchanged if
not sent or not observed. The owner accepts possible duplicates after a lost
acknowledgement. Report counts/minimal job references, never private content.
Do not change business data, templates, language or schedules outside this request.
```

## Phone lookup compatibility preview — awaiting deployment

Student, point-member and invoice-member searches are being enhanced to accept
formatted international numbers and local national-number suffixes. Prefer the
full `+` number when known. A local suffix can match several countries/accounts;
confirm the intended returned stable ID before any preview or write. If no
match is returned on the deployed server, refine by name/email through the same
MCP tool. Never invent a missing country code or bypass the MCP connection.
No new tool, argument, credential or permission is needed.

## Today's lesson reminders — deployed 17 November 2026

The lesson-template migration and server release are deployed. Update the
`tzh-lesson-reminders` skill in the client workspace and refresh MCP discovery.
Server enable/contact hours and supervised WhatsApp setup are still required.
Use **Lesson management** (`lessons:manage`) on an active admin's token. That
permission additionally allows private student contact/message preparation and
recording sent reminders; review existing token holders before rollout. No new
permission or automatic token grant is introduced.

Discover `list_lesson_reminders`, `prepare_lesson_reminder` and
`mark_lesson_reminder_sent`. Missing tools are a release/connection problem, not
permission to use SQL, source, scripts or internal HTTP as a fallback.

1. Admin → WhatsApp Reminders → **Class reminders** opens the editor directly
   at `/admin/reminders`. The old `/admin/whatsapp-reminders` URL redirects there;
   **Lesson fees** is separate CSV preparation. Save the EN/ZH/MS wording and review the fictional
   preview. Message language is independent of interface language.
2. The owner configures the independent lesson enabled/contact-hours gate on the
   server. Agree on the scheduler's language, time, authorized recipients, WhatsApp
   application and cap (at most 20 sends per run). Use one sender across reminder
   schedules. No schedule is installed or activated by editing templates.
3. After deployment, run the first supervised test with an owner-controlled
   recipient on a fictional enrollment in a class today that has not started.
   List, prepare, open the unchanged link, verify chat/prefill, send, observe the
   exact outgoing message, then mark sent with `confirm:true`.
4. Verify that another discovery/preparation skips the recorded enrollment.
   Only then activate the owner-approved schedule. Student absence, cancellation,
   started classes and invalid contacts must not receive reminders.

Read-only connection check:

```text
Use $tzh-lesson-reminders and list today's lesson reminders. Do not prepare,
open WhatsApp, send, mark sent or change a schedule. Report counts and skip reasons.
```

Example scheduler prompt, after supervised acceptance and explicit activation:

```text
Use $tzh-lesson-reminders to remind eligible students about today's class in
Asia/Kuala_Lumpur, using the saved English template. Process all eligible unsent
enrollments for today in this run, even if class is hours away. Do not wait for a
per-class due time or lead-time window. Send a separate reminder for each class
enrollment, including multiple classes for one student. Skip every non-null
skipReason, including started classes. I authorize matching reminder
sends through our approved WhatsApp session and recording observed sends. Run one
sender at a time and send at most 20 messages. Page discovery and prepare each
enrollment freshly immediately before sending. Stop on login, permission, handoff
or recipient/text mismatch. Inspect uncertain sends before retrying. Report sent,
skipped, failed and deferred counts without private contact/message/link details.
```

This is a class reminder, not a lesson-fee reminder. Registered students use their
account phone; guests use their enrollment phone. Billing/guardian contacts are
not automatically substituted. One observed send is recorded per enrollment,
regardless of selected language. A send with a lost acknowledgement can duplicate
on a later run; there is no reservation or delivery/read confirmation.

## Copy-and-paste scheduler setup prompts

Use the [stringing setup prompt](prompts/setup-stringing-reminder-scheduler.md)
or [daily lesson setup prompt](prompts/setup-lesson-reminder-scheduler.md) in the
updated customer workspace after the matching server release. These collect run
and follow-up timing interactively, check existing schedules and avoid overlapping
senders. They require server preparation gates and supervised WhatsApp readiness
before activating a schedule. They do not enable server contact hours themselves.
