# First-Use Setup and Preflight

Use this guide when the skill is being installed on a new computer or when the agent cannot find `lark-cli`, the official Lark companion skills, an authorized user identity, or a previously created tracker.

## Components

Keep these components separate in both explanations and permission decisions:

1. **The agent**: Codex, Claude Code, or another skill-capable coding agent.
2. **Lark CLI and its developer app**: lets the agent call Feishu/Lark APIs under an authorized identity.
3. **This skill**: provides the lab task-group schema and operating rules.
4. **The group-facing bot**: reads or sends group messages and may update the Base. The lab must create or connect this bot separately.

Installing this repository does not create a developer app, authorize a Feishu/Lark account, create a group bot, or grant the bot access to a Base.

The group-facing bot's displayed identity may differ from the developer app, CLI identity, service account, or other identity that actually writes Base records. Resolve and test the execution identity instead of granting access to whichever bot or app is easiest to find.

## Requirements

- Node.js 16 or newer, with npm/npx available.
- A Feishu/Lark account that may create or manage the required developer app and Base resources.
- A skill-capable agent such as Codex or Claude Code.
- Permission to create or manage the target group, Base, and sharing settings.
- A separately created or connected group bot when chat automation is required.

## Recommended Installation

Run the following in a terminal. Replace `<REPOSITORY_URL>` with the URL of this repository.

```powershell
npx @larksuite/cli@latest install
npx skills add larksuite/cli -g -y --skill lark-shared lark-base lark-drive lark-im lark-contact
npx skills add <REPOSITORY_URL> -g -y
```

The second command installs only the five required official companion skills: authentication, Base, Drive, group messaging, and member identity resolution. It does not install unrelated Lark modules. Installing these skills does not grant API scopes; actual access still depends on account authorization.

Restart the agent after installation so it discovers the new skills.

For a project-local installation, omit `-g`. A manual clone into the agent's skills directory is a fallback, but the skill installer is preferred because it validates the repository layout and discovers the skill automatically.

## Configure and Authorize Lark CLI

For an interactive user setup:

```powershell
lark-cli config init
lark-cli auth login --recommend
lark-cli auth status
lark-cli doctor
```

An agent creating a new developer app may use `lark-cli config init --new`, but browser confirmation and organization approval can still require the user. Request only the domains needed for this workflow: `base`, `drive`, `im`, plus `contact` when resolving member names, owners, or identity-based permissions.

Never ask a user to paste app secrets, access tokens, open IDs, or Base tokens into the repository or a public issue.

## Bot Write Access Has Three Layers

A bot is ready to update the tracker only when all three layers pass:

1. **App/API authorization**: the bot runtime has the message, Base, Drive, and identity scopes needed for its approved operations.
2. **Document and Base role**: the real execution identity is a Base collaborator and, when advanced permissions are enabled, belongs to a role that can edit `Task Register` and append `Update Log`.
3. **Runtime tool policy**: the bot platform or agent runtime exposes only the approved read/write tools. Base sharing does not create or change this allowlist.

For routine progress reports from group members, use a least-privilege runtime policy. Allow task lookup; confirmed updates to `Status`, `Latest Progress`, and internal source/snapshot fields; and append-only linked update-log writes. Deny task creation/deletion, owner/deadline/teacher-confirmation changes, other task fields, schema/views, Workflow, and permission administration unless a separate authorized workflow explicitly allows them.

Bind every write to the exact preview confirmed in the same conversation or message thread. Test through the group bot itself: first read one known task, then preview and confirm one harmless progress update, then verify both the task row and its linked audit row. Do not report readiness after testing only with the CLI account.

## Agent Preflight

Before creating any Feishu/Lark resource, the agent must report these checks:

| Check | Ready when |
|---|---|
| CLI | `lark-cli --version` and `lark-cli doctor` succeed |
| Companion skills | `lark-shared`, `lark-base`, `lark-drive`, and `lark-im` are discoverable |
| User identity | `lark-cli auth status` shows an authorized user for the intended tenant |
| Scopes | Base, Drive, and IM operations required by the requested setup are authorized |
| Existing resources | The target group and drive have been searched for a prior or partial setup |
| Group bot | The user has identified an existing bot or accepted that bot setup remains manual |
| Bot execution identity | The identity that actually performs group-triggered writes has been resolved |
| Bot Base access | That execution identity can edit `Task Register` and append `Update Log` under the active Base role |
| Bot runtime tools | The narrow progress-update allowlist is configured and a group-triggered read/write/log test passes |
| Approval mode | The user has chosen whether group messages and task writes require preview and confirmation |

If a check fails, stop at that checkpoint, explain the exact user action, and continue after it is resolved. Do not pretend that the full setup is ready.

## Resume Without Duplicates

Setup is resumable. On every run:

1. Search for the intended group by name and confirm its real `chat_id`.
2. Search the user's drive for the intended Base and inspect its tables.
3. Compare existing fields, views, roles, and Workflow with the template.
4. Create only missing items and repair incomplete items.
5. Read back every created or changed resource before reporting success.

Do not create a new group, Base, table, view, role, or Workflow solely because a previous run failed or the agent lost conversational context.

## Manual Checkpoints

Some steps may require the Feishu/Lark browser or desktop UI:

- Confirming or publishing a developer app.
- Completing user authorization or organization approval.
- Creating or connecting the group-facing bot and inviting it to the group.
- Granting advanced Base roles when the CLI cannot assign every role member.
- Configuring the bot platform's runtime tool allowlist; this is separate from Base sharing and may live in Aily, the custom bot service, or the connected agent platform.
- Configuring a paid or tenant-specific automation feature when Workflow capabilities differ.

The agent must distinguish completed API work from these remaining manual steps.

## Security Defaults

- Use least privilege and authorize only the required domains.
- Keep external sharing disabled unless the user explicitly requests it.
- Give ordinary members edit access to the task table only when requested.
- Keep the update log read-only for ordinary members.
- Do not expose a high-permission CLI identity or bot to untrusted groups.
- Require human confirmation before task creation, ownership or deadline changes, completion, deletion, and group-message sending when the user selects approval-first mode.
- For an ordinary group progress report, preview only task name, status, and latest progress. Do not infer teacher-confirmation items or unrelated questions.
- Bind confirmation to the exact preview and same conversation/thread; re-preview after any change.

## Troubleshooting

| Symptom | Action |
|---|---|
| The agent cannot find this skill | Restart the agent and verify the skill was installed globally or in the current project |
| `lark-cli` is not recognized | Install the official CLI, reopen the terminal, and run `lark-cli --version` |
| Base or IM commands are unavailable to the agent | Install the official Lark companion skills and restart the agent |
| `auth status` has no usable user identity | Run `lark-cli auth login --recommend` and complete browser/device authorization |
| A command reports missing scope | Authorize only the named domain, then retry the failed step |
| A rerun creates duplicates | Stop creation, search by name and IDs, and resume from the existing group/Base |
| The bot is in the group but cannot update Base | Identify the actual writing identity; verify app scopes, collaborator access, advanced Base role, and runtime write tools separately |
| The bot can read Base but says its tools are unavailable | Configure the bot/agent runtime allowlist; changing Base sharing alone will not expose tools |
| The bot writes too many fields or invents questions | Apply the minimal progress-update prompt and restrict runtime writes to status, latest progress, source/snapshot cleanup, and append-only audit logging |
| The bot says an update succeeded but the row did not change | Treat the run as failed, inspect runtime logs and permissions, then verify both the task row and linked update-log row before retrying |
| Manual edits are not logged | Verify the Workflow is enabled, its trigger fields and branches exist, and the tenant triggers Workflow for that edit source |
| CLI/API edits are not logged | Append the update-log row in the same confirmed CLI operation as the task change |

## Suggested First Prompt

```text
Use $lark-lab-task-group. Run the first-use preflight first. Verify the CLI,
official Lark companion skills, authorization, existing group/Base resources,
and group-bot availability. Show me any missing or manual steps. Do not create
or send anything until I confirm the setup summary.
```
