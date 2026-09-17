---
name: tzh-student-account-audit
description: Use the tzh_sports_centre MCP tools to find students and prepare, remove entries from, delete, review, or commit historical student-account audit cases. Trigger for student lookup, historical lesson or payment reconciliation, proof images, fee corrections, replacement corrections, draft cleanup, and audit case review or commit.
---

# TZH Student Account Audit

Use only the `tzh_sports_centre` MCP connection. Do not fall back to shell,
database, source-code, or direct HTTP access. If the server or a required tool
is unavailable, stop and report the connection problem.

## Available tools

- `search_student_accounts`: read-only search by name, email, or phone.
- `create_student_audit_case`: create a reversible draft for one confirmed
  student and a calendar year or explicit date range.
- `upload_student_audit_proof_image`: upload one private JPEG, PNG, or WebP
  proof image, up to 3 MiB decoded, to a confirmed draft case.
- `add_student_audit_entry`: add one reversible historical lesson, payment,
  fee-correction, or replacement-correction draft entry.
- `remove_student_audit_entry`: remove one confirmed entry from a draft case
  and clean up proof images no longer used by another entry.
- `delete_student_audit_case`: permanently delete a draft or rejected case and
  its owned proof images.
- `review_student_audit_case`: validate the draft and return its deterministic
  preview and validation version without applying it.
- `commit_student_audit_case`: apply an explicitly approved reviewed version
  to the canonical student account.

## Workflow

1. Search for the student even when the user supplies a name, email, or phone.
   Show the matching name and stable student ID. Stop on no match; ask the user
   to choose when more than one plausible record matches.
2. Before creating a case, confirm the stable student ID, case name, and either
   the calendar year or inclusive `YYYY-MM-DD` date range.
3. Create the draft with a unique idempotency key. Preserve the returned case
   ID for later calls.
4. Upload each proof image separately with its own idempotency key. Preserve
   the returned proof ID and attach it only to an entry or lesson exception in
   the same case.
5. Add one draft entry at a time. Confirm missing dates, amounts, weekday
   patterns, attendance statuses, descriptions, and correction signs instead
   of inventing them. Monetary inputs use integer cents.
6. Reuse the same idempotency key only when retrying the same logical write
   after an uncertain response. Use a new key for a different write.
7. If an entry is wrong, show its case ID and entry ID, obtain explicit
   confirmation, then call `remove_student_audit_entry`. To abandon the whole
   draft, show its case ID, obtain explicit confirmation, then call
   `delete_student_audit_case`. Never use either tool to undo a committed case.
8. Review after all intended entries are saved. Show the case details, every
   entry and its details, each attached evidence-image count, the canonical
   account position, audit fee/replacement changes, projected position, every
   warning and error, and the exact validation version. State clearly that
   canonical data is still unchanged. Evidence counts confirm attachment only;
   do not claim to have inspected or interpreted the private images.
9. Commit only after the user explicitly approves that exact preview and
   version. Pass `confirm: true` and the reviewed validation version. Never
   silently review a newer version and commit it under earlier approval.

## Guide the next step

After each completed stage, briefly state the outcome and whether canonical
student data changed. If the user already requested additional safe draft work
and supplied everything needed, continue it. Otherwise end with one focused
question based on the current state:

- After search, ask the user to choose or confirm the stable student ID and the
  intended year/date range.
- After case creation, ask which historical lesson, fee payment, correction, or
  proof image to add first.
- After proof upload, ask which entry the proof supports or whether to upload
  another proof.
- After adding or removing an entry, ask whether to continue authoring, manage
  proof, or review the complete case.
- After a failed review, show every error and ask which draft entry to correct
  or remove. Never invent the correction.
- After a successful review, show the complete web-equivalent account-effect
  preview and exact validation version, then ask whether the user explicitly approves committing that exact
  version or wants to stop. This question is not approval.
- After commit, report completion and ask whether to start another audit or
  finish.

Do not ask a vague “What next?” when the valid choices can be named. Do not
infer missing facts, select a student, remove data, delete a case, or commit
from silence or from a generic request to continue.

## Entry rules

- Historical recurring lessons require a class label, fee in cents, duration,
  one or more weekdays, and any known per-date attendance exceptions.
- Historical payments require a positive amount in cents and use the case end
  date as the recorded payment date.
- Fee corrections require a non-zero signed amount in cents, effective date,
  description, and optional reason. Confirm whether the correction increases
  or decreases the account balance.
- Replacement corrections require a non-zero signed quantity. Confirm whether
  the correction adds or removes replacements.
- Chat-derived facts remain drafts until the reviewed version is explicitly
  approved and committed.

## Results and failures

- State which tool ran and whether the outcome was read-only, draft, reviewed,
  or committed.
- Include stable student, case, entry, and proof IDs only when needed for the
  next step. Never include credentials.
- Surface tool errors faithfully. For an uncertain commit result, retry only
  the exact same case ID and explicitly approved validation version. Never use
  a newer version without a new review and approval.
- HTTP 413 means the proof request exceeded the platform payload limit; ask for
  a smaller image. HTTP 504 means the function timed out; report it. Retrying
  the exact approved commit is safe and does not repeat canonical records,
  invoices, or emails.
