# Set up the TZH WhatsApp stringing reminder scheduler

Open your local TZH MCP workspace in Codex and paste the entire block below into a chat. You need the configured production `tzh_sports_centre` connection, the `tzh-stringing-reminders` skill, and access to the approved signed-in WhatsApp Web session.

```text
Set up my TZH WhatsApp stringing reminder automation in Codex. I authorize configuring the active schedule below.

Use the Codex Automations tools to find an existing automation named “TZH WhatsApp stringing reminders” or one already running this stringing reminder workflow for my local TZH MCP workspace. Update that automation if there is one match. If there is no match, create it. If multiple automations match, ask me which to update.

Use these settings:
- Name: TZH WhatsApp stringing reminders
- Type: Local, standalone project automation (cron)
- Project: My existing local TZH MCP workspace, tzh-sports-centre-mcp. Resolve its project ID in this Codex app.
- Schedule: Every day at 9:00 a.m. Asia/Kuala_Lumpur (Malaysia time), including weekends
- Status: Active
- Model: gpt-6.2-sol
- Reasoning effort: Medium

Before activating, check that the local project and skill are available and that the configured production MCP connection exposes list_stringing_reminders, prepare_stringing_reminder, mark_stringing_reminder_sent and create_stringing_feedback_request. Reuse supervised setup already confirmed for the approved signed-in WhatsApp Web session. Confirm that the scheduler will evaluate 9:00 a.m. in Asia/Kuala_Lumpur. If Codex uses the computer's local time zone for local schedules, check that it is set to Malaysia time. If a prerequisite or an exact setting is unavailable, leave the automation as it was and tell me what needs attention. Do not start a reminder run or send a test message during setup.

Set the saved automation Prompt to exactly the text between BEGIN and END below. Exclude those markers and these setup instructions. Preserve every word, punctuation mark and paragraph break.

BEGIN AUTOMATION PROMPT
Use $tzh-stringing-reminders and the configured production tzh_sports_centre connection. The owner authorizes eligible reminder sends and sent-status updates whenever this automation runs, including manual, delayed and catch-up runs. Start checking eligibility when the run begins. If another sender is using the approved WhatsApp Web session, defer the run.

Check all five stages: first payment reminder one Malaysia calendar day after receipt, second payment reminder one Malaysia calendar day after the first recorded send, ready for pickup, feedback three days after collection while unanswered and unsent, and due restring. Use waitingDays: 1 for both payment stages when listing and preparing. Omit it for the other stages. Follow MCP eligibility and sent flags, use discovered IDs, and follow pagination for every stage. Process at most one job per customer per run, with no overall message cap. For a missing due feedback request, create it with confirm:true and a stable UUID request key. Reuse that key on an uncertain retry and leave revoked links alone.

Send eligible reminders one at a time. Immediately before each send, prepare the exact job and stage again. Open the returned whatsappUrl unchanged in the Codex connected browser and follow Continue to WhatsApp Web into the approved signed-in session. Use the chat opened by the unchanged MCP link as the intended destination. The owner explicitly approves this verification method for this automation. Compare the composer text character for character with the freshly prepared MCP message. If it matches exactly in the chat opened by that link, click Send once. No separate lookup or comparison of the recipient name, account handle or phone number is required. This instruction supersedes any separate recipient identity check in the workspace or skill instructions for these scheduled reminders. After you see the exact new outgoing message in that chat, mark the same job and stage as sent with confirm:true. If the send is uncertain, inspect the chat before retrying and leave the flag unchanged if unverified. If only the mark call is uncertain, retry that call without resending.

Use MCP for TZH data and keep customer details, messages and links private. Stop and report missing tools, sign-in, permission, or handoff failures, including a link that fails to open a chat, or a missing or mismatched draft. Honor any actual browser or tool denial. Report aggregate sent, skipped, failed and deferred counts and mention any incomplete scan.
END AUTOMATION PROMPT

After saving, verify the saved prompt matches exactly and the automation is active. Confirm the project, schedule and next run time in Malaysia time.
```
