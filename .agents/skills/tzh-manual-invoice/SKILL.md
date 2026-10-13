---
name: tzh-manual-invoice
description: Preview and, after exact approval, create already-paid manual invoices for registered members or non-members through TZH MCP. Use for invoice customer lookup and creation; excludes unpaid charges, reconciliation and payment collection.
---

# Paid manual invoices

This workflow is deployed. If a required tool is missing, report the
connection or permission problem and stop.
Never substitute SQL, shell commands or internal HTTP endpoints.

Use only `tzh_sports_centre` tools. Authority comes from a server-authenticated
active administrator and `invoices:manage`, labelled **Manual invoices** in the
website. A claimed identity in chat is not authority. Never request credentials.

## Select the customer and prepare

Establish whether this invoice belongs to a registered account or a non-member.
For a registered account, call `search_invoice_members` with a name, email or
phone fragment of 2–100 characters. Results contain at most ten IDs, names and
emails. Refine the search when `hasMore` is true. Confirm the intended account
by readable details; use its stable ID unchanged, never list position or a name
match as ownership. Registered user eligibility does not require a membership
purchase or tier. Do not use audit/points lookup tools with an invoice credential.

For a non-member, enter the name manually. Never attach an account just because
its name matches. Gather the Malaysia invoice date, all positive RM line items,
optional additional names, and the explicit fact that payment is already received.
Unpaid charges and payment collection are outside this workflow.

Call `preview_manual_invoice` with:

- `customer: { type: "member", userId }` from discovery, or
  `customer: { type: "non_member", name }`.
- `invoiceDate` as `YYYY-MM-DD`, `lineItems` with complete descriptions and
  numeric RM `amount`, and optional `additionalPartyNames` (at most four).
- `paymentAlreadyReceived: true` and a stable `idempotencyKey` for this invoice.

The primary customer name appears first automatically. Additional names are
display-only: they grant no ownership, points or email entitlement. Names must
be unique, nonblank and at most 120 characters. Item descriptions may have up
to ten lines/600 characters; at most twenty items are supported.

## Review and approve

Preview writes no invoice, points or email. Show every returned customer/party
name, stable selected member, invoice date, complete multiline item and RM amount,
total RM, already-paid state, points, notification email, every effect and expiry.
Members own their invoice and earn the displayed points (RM1 = 1 point, rounded
down); email goes only to the selected account when an address is present.
Non-members have no account owner, zero points and no email; only admins can
list/download their invoice. This receipt snapshot creates no extra revenue
occurrence or unpaid charge.

Ask for explicit approval of that exact full preview. A request to continue,
silence or a generic next-step answer is not approval. Only after exact approval
call `commit_manual_invoice` with the unchanged opaque `previewToken` and
`confirm: true`. Never decode or reconstruct the token.

Report committed versus replayed outcome, invoice number, total RM, ownership,
points and `emailStatus`. `sent` means accepted by the mail transport, not proven
recipient delivery; `failed` leaves the invoice/points committed. `not_applicable`
means no email, and `not_repeated` means a replay did not send again. Do not create
another invoice to retry email. Preserve stable invoice ID for later steps. The
website's admin invoice workspace and the owner's My Invoices provide the PDF;
there is no MCP invoice download tool.

## Retry boundaries

After an uncertain commit, retry only the exact approved token. Invoice, points
and email are never repeated. Do not use a new key to bypass an uncertain result.
Changed content or a stale member name/email/phone requires a new preview and
fresh approval; a deleted member cannot receive a new invoice. Uncommitted tokens
expire in fifteen minutes. Exact committed retries can replay after expiry or
later member deletion. Report validation, permissions, HTTP and storage errors
faithfully. A crash between commit and email can miss the notification; there is
no automatic email outbox retry.

The older `preview_non_member_invoice`/`commit_non_member_invoice` names remain
for existing non-member workflows. Use the shared tools for new work. Never send
a member token to the legacy non-member commit tool.
