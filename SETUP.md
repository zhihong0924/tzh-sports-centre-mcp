# Set up the TZH Sports Centre MCP workspace

Keep this Git repository intact. It contains:

```text
tzh-sports-centre-mcp/
├── AGENTS.md
├── SETUP.md
└── .agents/
    └── skills/
        └── tzh-student-account-audit/
            └── SKILL.md
```

`AGENTS.md` supplies instructions for every task started from this workspace.
Codex discovers the skill from `.agents/skills`; it reads the full `SKILL.md`
only when the request matches the skill or you invoke
`$tzh-student-account-audit` explicitly. A loose file named `skills.md` is not
the Codex skill format.

## 1. Store the bearer token on macOS

TZH supplies the token separately through a secure channel. A TZH administrator
normally creates a separately named, independently revocable token from the
website's **MCP Access Tokens** workspace; its plaintext is shown only once.
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
2. Enter `/skills`, or type `$`, and confirm `tzh-student-account-audit` appears.
3. Run this read-only smoke test with a safe search value:

```text
Use $tzh-student-account-audit and call search_student_accounts from
tzh_sports_centre with {"query":"EXAMPLE NAME OR EMAIL"}. Do not create or
modify anything.
```

The expected result is a bounded list of matching active students or an empty
list. If several students match, continue only after choosing by stable student
ID.

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

## 5. Troubleshooting and updates

- If the server is absent, check the exact URL, environment variable name, and
  `[mcp_servers.tzh_sports_centre]` table, then fully restart Codex.
- If the skill is absent, confirm the hidden `.agents` directory was included,
  the file is exactly `.agents/skills/tzh-student-account-audit/SKILL.md`, and
  this folder is the project's main folder. Start a new task after correcting it.
- Never paste a token into chat while troubleshooting. Ask TZH to revoke and
  replace any token that may have been exposed.
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
