---
name: tzh-point-assignment
description: Use the tzh_sports_centre MCP tools to find active members and award positive points through active policies or one free-form reason. Trigger for member point awards, bonuses, policy-based points, or previewing and committing a point assignment. Do not use for deductions, fee reminders, or Student Account Audit reconciliation.
---

# TZH Point Assignment

Use only the `tzh_sports_centre` MCP connection. Do not fall back to shell,
database, source-code, or direct HTTP access. This workflow awards positive
points only; report deductions and fee reminders as unsupported.

## Workflow

1. Call `search_point_members` for each intended person even when the user
   supplies a name or email. Select only returned stable member IDs. Stop on no
   match; if a search is ambiguous, show readable identity details and ask the
   administrator to choose. Never select by list position or chat claim.
2. For policy mode, call `list_active_point_policies` and use only returned
   stable policy IDs. Fixed policies omit `amountCents`. Every money-based
   policy requires its own positive integer-cent amount supplied or confirmed
   by the administrator.
3. Use exactly one assignment mode:
   - policy mode: one or more distinct members and active policies;
   - free-form mode: one or more distinct members, a positive integer point
     amount, required reason, and optional note.
   Never mix policy selections with a free-form award.
4. Generate one new idempotency key for the logical assignment and call
   `preview_points_assignment`. Reuse that key only for an uncertain retry of
   unchanged content.
5. Show the complete server preview: every member, every policy or free-form
   contribution, points per member, member count, ledger-row count, total
   points, and preview expiry. State that no points have been written.
6. Ask whether the administrator explicitly approves that exact preview. A
   request to continue, finish, or take the next step is not approval.
7. Only after explicit approval, call `commit_points_assignment` with
   `confirm: true` and the exact opaque `previewToken`. Never edit, decode,
   reconstruct, or substitute the token. If the preview is expired or stale,
   preview current data again, show it in full, and request new approval.

## Results and failures

- Report which tool ran and whether the outcome was discovery, previewed, or
  committed. A preview never changes the ledger.
- An identical commit retry can return the original operation with
  `replayed: true`; report that no duplicate rows were written.
- If a commit result is uncertain, retry only the same opaque preview token.
- If authentication is missing `points:manage`, ask TZH for a point-enabled
  replacement token through a secure channel. Never ask the user to paste a
  token into chat.
- Preserve stable IDs only as needed for the workflow and never expose bearer
  credentials or internal diagnostics.
