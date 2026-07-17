# First-Use Setup and Preflight

Use this guide when the skill is being installed on a new computer or when the agent cannot find `lark-cli`, the official Lark companion skills, an authorized user identity, or a previously created tracker.

## Components

Keep these components separate in both explanations and permission decisions:

1. **The agent**: Codex, Claude Code, or another skill-capable coding agent.
2. **Lark CLI and its developer app**: lets the agent call Feishu/Lark APIs under an authorized identity.
3. **This skill**: provides the lab task-group schema and operating rules.
4. **The group-facing bot**: reads or sends group messages and may update the Base. The lab must create or connect this bot separately.

Installing this repository does not create a developer app, authorize a Feishu/Lark account, create a group bot, or grant the bot access to a Base.

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
- Configuring a paid or tenant-specific automation feature when Workflow capabilities differ.

The agent must distinguish completed API work from these remaining manual steps.

## Security Defaults

- Use least privilege and authorize only the required domains.
- Keep external sharing disabled unless the user explicitly requests it.
- Give ordinary members edit access to the task table only when requested.
- Keep the update log read-only for ordinary members.
- Do not expose a high-permission CLI identity or bot to untrusted groups.
- Require human confirmation before task creation, ownership or deadline changes, completion, deletion, and group-message sending when the user selects approval-first mode.

## Troubleshooting

| Symptom | Action |
|---|---|
| The agent cannot find this skill | Restart the agent and verify the skill was installed globally or in the current project |
| `lark-cli` is not recognized | Install the official CLI, reopen the terminal, and run `lark-cli --version` |
| Base or IM commands are unavailable to the agent | Install the official Lark companion skills and restart the agent |
| `auth status` has no usable user identity | Run `lark-cli auth login --recommend` and complete browser/device authorization |
| A command reports missing scope | Authorize only the named domain, then retry the failed step |
| A rerun creates duplicates | Stop creation, search by name and IDs, and resume from the existing group/Base |
| The bot is in the group but cannot update Base | Share the Base with the bot's execution identity and verify Base edit permission |
| Manual edits are not logged | Verify the Workflow is enabled, its trigger fields and branches exist, and the tenant triggers Workflow for that edit source |
| CLI/API edits are not logged | Append the update-log row in the same confirmed CLI operation as the task change |

## Suggested First Prompt

```text
Use $lark-lab-task-group. Run the first-use preflight first. Verify the CLI,
official Lark companion skills, authorization, existing group/Base resources,
and group-bot availability. Show me any missing or manual steps. Do not create
or send anything until I confirm the setup summary.
```
