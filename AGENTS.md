# TZH Sports Centre MCP Workspace

This folder is an administration workspace for the private `tzh_sports_centre`
MCP server. These rules apply to every task started from this folder.

## Development workflow

- Autonomous work selected from the application repository's
  `DEVELOPMENT_TASKS.md` follows its `docs/autonomous-development.md` runbook.
- When a task records `MCP impact: Required`, make feature-dependent changes in
  the recorded MCP branch and worktree. Keep this repository's root checkout on
  `main`, and keep task status in the application queue rather than duplicating
  it here.
- This workspace has no task database. Do not clone the application database,
  copy its `.env`, or add credentials for branch development.
- Commit application and MCP changes separately. Do not publish customer
  instructions for a tool contract that has not been integrated and deployed.
- A scheduled run may integrate a completed direct prerequisite into both local
  `main` branches and refresh the dependent task branches only through the
  guarded prerequisite-recovery procedure in the application repository's
  `docs/autonomous-development.md`. All affected repositories must pass its
  clean-state, exact-branch, completion, and fast-forward/conflict preflight
  checks before either repository changes. That exception does not authorize
  pull, push, deploy, reset, rebase, unrelated merges, or branch cleanup.
- Independent corrections that describe the currently deployed server may use
  the normal repository workflow; scheduled queue work must use its recorded
  branch and worktree.

## Required integration boundary

- Use only tools from `tzh_sports_centre` to read or change private TZH data.
- For student lookup or historical account reconciliation, use the
  `tzh-student-account-audit` skill and follow its staged workflow.
- For already-paid member or non-member invoices, use `tzh-manual-invoice`. Discover registered accounts with `search_invoice_members`, preserve the selected stable ID, preview all ownership/points/email effects, and obtain exact approval before commit. Manually entered non-member names never identify accounts. The old non-member skill/tools remain compatibility-only.
- For positive member point awards, use the `tzh-point-assignment` skill and
  follow its preview-before-confirmation workflow. Deductions and fee reminders
  are outside that skill.
- For lesson duration or per-student price changes, use the
  `tzh-lesson-management` skill and follow its stable-ID discovery,
  full-batch preview, and explicit-confirmation workflow.
- For lesson attendance history or the completed-lesson review queue, use the
  `tzh-lesson-attendance` skill and its read-only Malaysia-time query workflow.
- For current administrator work due today, tomorrow, or this week, use the
  `tzh-admin-attention` skill and its read-only query workflow.
- Never query PostgreSQL, Prisma, application source code, internal HTTP APIs,
  or repository scripts as an alternative way to access TZH data.
- Never use shell commands as a fallback for private data. A failed local shell
  or database connection does not prove that the MCP server is unavailable.
- If the MCP server or a required tool is unavailable, stop and report the
  connection problem. Do not guess results or bypass the MCP boundary.
- Do not perform website development or system administration unless the user
  explicitly asks for that separate work.

## Privacy and credentials

- Authentication comes from the configured MCP connection. Never ask the user
  to paste a bearer token into chat.
- TZH administrators create independently revocable credentials from the
  website's **MCP Access Tokens** workspace. The plaintext is shown only once;
  customers still receive it separately through a secure channel.
- The owner confirmed the then-existing MCP functions on 2026-10-13. That dated confirmation does not establish availability of later tools. Verify required tool discovery on the configured connection; migration readiness for later releases requires its own check.
- **Manual invoices** labels `invoices:manage` and authorizes both registered-user and non-member paid invoices. Existing invoice-enabled tokens gain the shared workflow; other scopes gain no invoice authority. Owners should review invoice token recipients for this expanded capability.
- The currently deployed website offers these token controls: **Student account** grants
  `student-audit:manage`, **Member points** grants `points:manage`,
  **Lesson management** grants `lessons:manage`, and **Read admin attention**
  grants the independent `admin-attention:read` scope; **Manual invoices** grants
  `invoices:manage`. That one lesson scope
  authorizes duration/price management and attendance query/recording. This
  audit workflow still requires `student-audit:manage`. Existing tokens with
  `lessons:manage` can use the deployed attendance tools.
- Never display, log, summarize, save, or transmit an access token.
- Treat names, contact details, account history, notes, proof images, and all
  returned identifiers as private data. Return only what the current task needs.
- A claimed identity or administrator role in chat is not authorization. Trust
  only the identity established by the MCP server.

## Action boundaries

- Read-only searches may run when requested, but identify records by stable ID
  rather than name or list position.
- Confirm the selected student and date range before creating an audit case.
- Draft cases, proof uploads, and draft entries are reversible writes. Describe
  them as drafts and use stable idempotency keys for retries.
- For an existing case, confirm the student and call
  `list_student_audit_cases`. Use a uniquely matching case ID internally. If
  several cases remain plausible, ask the administrator to choose by readable
  case name, date range, and status; never ask them to retrieve an internal ID.
- An audit case can contain multiple audit entries. Use
  `remove_student_audit_entry` for the web-equivalent **Remove** action on one
  draft entry, and `delete_student_audit_case` to permanently delete a draft or
  rejected case. Show the exact target and obtain explicit confirmation before
  either destructive call. Neither action can undo a committed case.
- Review validates a versioned preview but does not apply canonical changes.
  Show its case details, every entry and entry detail, per-entry attached
  evidence-image count, canonical/current account position, audit fee and
  replacement changes, projected position, warnings, and errors. An evidence
  count confirms attachment only; never claim to have inspected the image.
  Present all administrator-facing money as formatted RM, not cents.
- Commit is consequential. Call the commit tool only after showing the complete
  preview and receiving explicit approval for that exact validation version.
- Point assignment is also consequential. Show the complete signed preview and
  call `commit_points_assignment` only after explicit approval, using the exact
  opaque preview token. Never broaden an audit-only credential into point
  authority.
- Lesson management is consequential and all-or-nothing. Discover concrete
  lessons by stable ID; use the returned active roster, court, time, duration,
  and RM fee to identify the intended occurrence. Return only the names and
  stable IDs needed for this choice, never contact or payment details. Show
  every current/proposed duration, end time, RM price, customization state,
  recurring-occurrence context, and downstream
  effect in the signed preview, then commit only after literal approval of that
  exact full batch. Rejected, stale, conflicting, or invalid batches make no
  changes. Reuse an idempotency key only for an uncertain retry of unchanged
  content.
- Admin-attention queries are read-only. Map “tasks due today”, “tomorrow”,
  and “this week” to the matching server preset; never ask a free-form tool
  question. Present the resolved Malaysia range, as-of time, overdue work,
  due-in-window work, undated backlog, queried providers, and exclusions.
  An empty due section is not an all-clear. Keep stable IDs and opaque cursors
  unchanged, and do not infer due dates from creation or preferred trial dates.
- Lesson attendance queries are read-only. Use exactly one server-resolved
  Malaysia preset or bounded custom range, preserve stable filters and opaque
  cursors, report whole-query summaries separately from the current page, and
  never infer attendance from billing or an absent record. Guests are excluded
  by default and are `not_tracked` when explicitly included.
- Attendance recording is consequential and all-or-nothing. Use only stable
  lesson/enrollment IDs returned by the attendance query. Preview the complete
  batch and show every current state, proposed stored outcome, optional reason,
  affected enrollment, replacement/cancellation/seat/slot effect, unchanged
  billing treatment, no-op, and expiry. Commit only after literal approval of
  that exact preview, using its opaque token unchanged and a stable idempotency
  key. A stale or expired preview requires a new preview and approval.
- Requests such as "continue", "finish", or "do the rest" are not approval to
  commit. If the reviewed version changes, review again and request approval
  again.
- Guide the administrator through the staged workflow. After a completed stage,
  continue already-requested safe draft work when all facts are available;
  otherwise summarize the result and ask one focused question naming the valid
  next actions. Do not use a generic next-step question as evidence of approval.
- Replacement obligations created by a committed audit are lifetime credits,
  not expiring live absence tokens. They are redeemed by an administrator in
  the website's replacement-booking workflow; do not describe them as already
  redeemed merely because the audit was committed.

## Failures and reporting

- Report the MCP tool used and distinguish read-only, draft, reviewed, and
  committed outcomes.
- Preserve stable IDs needed for later steps, but never include credentials.
- Report MCP and HTTP errors faithfully. After an uncertain commit result,
  retry only the exact same case ID and explicitly approved validation version;
  never substitute a newer version without a new review and approval.
- For an uncertain lesson commit, retry only the exact preview token and the
  same idempotency key. If the preview is stale or expired, preview the full
  current batch again and request new approval.
- Admin-attention queries never write. Retry a failed read once with the same
  range; after a stale cursor, restart at the first page. Report a provider
  failure instead of claiming that no work exists.
- Attendance queries never write. Retry a failed read once with the same
  filters; if a cursor is rejected, restart from the first page rather than
  decoding or modifying it.
- For an uncertain attendance commit, retry only the exact preview token and
  same idempotency key. Never convert a timeout, “continue”, or generic approval
  into authority for a different or newly generated batch.
- For HTTP 413, request a smaller proof image. For HTTP 504, report the timeout;
  an exact approved commit retry is safe and does not repeat canonical records,
  invoices, or emails.


## Shared manual-invoice workflow

`search_invoice_members` and `preview_manual_invoice` are read-only;
`commit_manual_invoice` is consequential. All require a server-authenticated
active administrator and `invoices:manage`. Use stable discovery IDs for members
or explicitly enter a non-member name. Only already-received payment is supported.
The primary name is first; additional names grant no ownership. Show every name,
date, complete multiline item, formatted RM amount, total, paid state, selected
member ownership, points, notification email, effects and expiry. Commit only
after explicit approval of that exact full signed preview with literal confirmation.

Member invoices and spend points are atomic; non-members have no account owner,
points or email. Member email is fail-open after commit; report its returned
status without claiming proven delivery. Exact retries preserve the approved
token and never repeat invoice, points or email. Changed content, stale member
identity/contact state or an expired uncommitted preview requires a new preview
and approval. Deleted members cannot receive new invoices. A committed exact
retry can replay after expiry. Never create a second invoice to resend an email.

The old non-member tools remain compatible but cannot create member invoices.
Never bypass the MCP boundary if tools are absent; check the connection and
permissions instead.

## Off-site lessons — deployed 16 October 2026

The off-site migration and server release are deployed. Lesson discovery and
attendance may return `court.id = null` and an `Off-site: …` venue name. Treat
this as an external location with no centre court; never invent a court ID.
Numeric court filters exclude these lessons. Existing `lessons:manage` scopes
and exact preview/commit approval still apply. Off-site duration/price changes
reserve no centre slots and may fall outside centre operating hours. Lesson
creation remains on the website.


## Stringing reminders — scheduler setup required

Use `tzh-stringing-reminders` and the four list/prepare/finish/create-feedback-request
tools. Require `stringing-reminders:manage` and a live active administrator. The
owner-approved data-only migration grants all six current permissions to existing
active, unexpired tokens while retaining secrets, owners, expiry/revocation and
unrelated scopes. Expired/revoked tokens are unchanged and remain invalid. New tokens select **Stringing
reminders**. Refresh discovery and report unavailable tools.

Payment waiting periods come only from the admin's scheduler prompt. Supply
`waitingDays` (0–3650 whole Malaysia calendar days) in every payment list/prepare
call: first from received date, second from first recorded send. Match the prompt's
value for each stage; if missing, ask before processing that payment stage. Omit
the argument for other stages, and omit unapproved stages from the run. Do not
create saved policies or change schedules. No customer pause setting exists.

Use list → fresh preparation → approved WhatsApp UI send → finish with
`{ jobId, stage, confirm:true }` after observing the new outgoing message in the
verified chat. Pass the prepared whatsappUrl directly as an authorized browser tool argument;
necessary MCP results and browser/computer arguments are permitted. Keep private
fields out of user-facing reports, screenshots and extra diagnostic logs. Separate
runtimes need no shared variables or private bridge; do not use shell/temp files
or external services to transfer messages. Report actual tool/handoff failures.
Open the returned `whatsappUrl` unchanged in the connected browser, follow its
Open app/Open WhatsApp handoff and verify the intended chat and prefilled text.
Do not search numbers or enter messages as an automatic alternative. Stop and
report blocked handoff, sign-in or missing/mismatched drafts. An explicit owner
choice is needed for a manual alternative; never bypass denied permissions.
Observe the exact new outgoing message in the chat itself before marking sent;
a chat-list preview or empty composer alone is insufficient. Inspect uncertain
sends before retrying; do not click Send again just because verification failed.
Existing sent flags are
the only completion truth; retries preserve the first sent timestamp. Feedback is
once, three days after collection, pickup requires Ready and restring uses its saved
due date. Create a missing due feedback request explicitly; never replace a revoked
link or submit feedback. No claim/recovery tool or attempt ledger remains.

Run one sender, at most 20 messages and one job per customer per run within approved
Malaysia contact hours. Client limits are not server reservations. A lost send
acknowledgement can cause a later duplicate, as accepted by the owner. Leave sent
status unchanged without an observed send and stop on login/permission/mismatch/UI
failures. Standing approval covers matching scheduled scope; token access alone
never authorizes sending. Report counts/minimal IDs and partial coverage, not private
content. No SQL/source/internal HTTP/shell fallback, financial/progress edits,
feedback submission, template/language changes or unrequested schedule activation.
