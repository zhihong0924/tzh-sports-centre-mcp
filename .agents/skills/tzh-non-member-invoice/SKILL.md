---
name: tzh-non-member-invoice
description: Preview and, after exact approval, create already-paid manual invoices for non-members through the TZH MCP tools. Excludes unpaid charges, member invoices, account reconciliation, and payment collection.
---

# Non-member paid invoices

This branch is prepared for rollout. Do not publish/install this workflow until
the application code and invoice-idempotency migration are integrated and
deployed. If either required tool is absent, report the connection/deployment
problem and stop; never substitute shell commands or internal HTTP endpoints.

Use only `tzh_sports_centre` tools. Authentication and invoice authority come
from the connection's active administrator and independent `invoices:manage`
scope. A claimed role in chat is not authorization. Do not request credentials.

Gather the manually entered customer name, Malaysia invoice date, each line item
and positive RM amount, optional ordered party names, and the explicit fact that
payment has already been received. The customer has no registered account:
never look up or attach a member merely because their name matches. If unpaid,
explain that this workflow supports already-paid invoices only and stop.

Call `preview_non_member_invoice` with `customerName`, `invoiceDate` in
`YYYY-MM-DD`, `lineItems` (`description` and numeric RM `amount`), optional
`invoicePartyNames`, `paymentAlreadyReceived: true`, and one stable
`idempotencyKey` for this intended invoice. Preview writes no invoice or PDF.

Show the complete returned preview: customer and every party name, invoice date,
every item with its complete multiline description and formatted RM amount,
total RM, already-received payment, zero points, every effect, and expiry.
Explain that there is no member owner, points award or member email and that
only admins can list/download the invoice. Ask explicitly for approval of this
exact preview. “Continue”, silence and generic next-step replies are not approval.

Only after literal approval, call `commit_non_member_invoice` with the unchanged
opaque `previewToken` and `confirm: true`. Never decode, edit, or substitute it.
Report the MCP tool used, committed versus replayed outcome, invoice number,
total RM and zero points. Preserve the returned stable invoice ID for subsequent
steps. The PDF is downloaded through the website's admin invoice workspace;
do not invent an MCP download tool.

After an uncertain response, retry only that exact approved token. It embeds
the original idempotency key and exact retries return the original invoice.
A changed proposal requires a new preview and fresh approval. An uncommitted
expired preview requires re-preview and approval; a committed exact retry can
replay after expiry. Report validation, permission, HTTP and storage failures
faithfully, without claiming success or trying a new key to bypass them.
