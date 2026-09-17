# TZH Sports Centre MCP Workspace

This folder is an administration workspace for the private `tzh_sports_centre`
MCP server. These rules apply to every task started from this folder.

## Required integration boundary

- Use only tools from `tzh_sports_centre` to read or change private TZH data.
- For student lookup or historical account reconciliation, use the
  `tzh-student-account-audit` skill and follow its staged workflow.
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
- Review validates a versioned preview but does not apply canonical changes.
- Commit is consequential. Call the commit tool only after showing the complete
  preview and receiving explicit approval for that exact validation version.
- Requests such as "continue", "finish", or "do the rest" are not approval to
  commit. If the reviewed version changes, review again and request approval
  again.

## Failures and reporting

- Report the MCP tool used and distinguish read-only, draft, reviewed, and
  committed outcomes.
- Preserve stable IDs needed for later steps, but never include credentials.
- Report MCP and HTTP errors faithfully. After an uncertain commit result,
  retry only the exact same case ID and explicitly approved validation version;
  never substitute a newer version without a new review and approval.
- For HTTP 413, request a smaller proof image. For HTTP 504, report the timeout;
  an exact approved commit retry is safe and does not repeat canonical records,
  invoices, or emails.
