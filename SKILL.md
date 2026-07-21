---
name: lark-lab-task-group
description: "Build a Feishu/Lark research-lab task group from zero to one using lark-cli: create or locate a group chat, add/brief a bot, create a Base task tracker, configure fields/categories/views/permissions, post the tracker to the group, and set lightweight rules for task intake, progress updates, teacher confirmation, weekly reminders, and summaries. Use when researchers want a reusable lab task-management template in Feishu/Lark, especially for student-supervisor groups, wet-lab workflows, sample submission tracking, planting/experiment schedules, and multi-person progress coordination."
metadata:
  requires:
    bins: ["lark-cli"]
    siblings: ["lark-shared", "lark-base", "lark-drive", "lark-im", "lark-contact"]
  cliHelp: "lark-cli --help"
---

# Lark Lab Task Group

## Purpose

Set up a lightweight lab task workflow in Feishu/Lark:

- Group chat for discussion.
- Base task tracker for status and accountability.
- Bot for organizing chat content, reminding people, and producing weekly summaries.
- Clear rules for when people update the Base directly versus when the bot asks for confirmation.

Always protect privacy. Do not include real gene names, sample names, unpublished project names, people's names, or institution-specific details in reusable templates. Use neutral examples such as `Genotype A`, `Material group`, `Student A`, and `Assistant`.

## Collaboration Model

Use this operating model by default unless the user chooses another one:

- Do not assume the lab has a fixed coordinator. Any authorized member may capture fragments, request a draft, review it, and confirm a write.
- Treat the private bot conversation primarily as a lightweight fragment-capture channel for tasks, progress, and open questions arising online or offline.
- Treat Codex plus `lark-cli` as the primary path for turning natural-language context into a written table draft and for confirmed batch creation or updates.
- Treat the group chat as the transparent team channel for assignments, routine progress, blockers, questions, and decisions.
- Treat `Task Register` as the single source of truth for current task state.
- Treat `Update Log` as an append-only audit trail for ordinary members.
- Use a draft-first loop before every agent-assisted task write. Send group messages only according to the user's chosen approval mode.
- Also let members edit `Task Register` directly or report progress by mentioning the bot. Archive every path through Workflow or an explicit bot/CLI log write.
- For an existing-task progress report made by mentioning the bot, use the minimal progress-update protocol: identify one task, preview only `Task Name`, proposed `Status`, and proposed `Latest Progress`, then ask only whether to upload. Do not turn routine progress into a new task or a new teacher-confirmation question.
- Before a weekly report, ask members to maintain current-stage deadlines. Build the report only from the three validated dynamic views.

## Workflow

0. Run the first-use preflight and resume safely:
   - Read `references/getting-started.md` when the user is installing this skill for the first time, `lark-cli` is missing, authentication is incomplete, or required companion skills are unavailable.
   - Verify Node.js/npm, `lark-cli`, and the official companion skills `lark-shared`, `lark-base`, `lark-drive`, `lark-im`, and `lark-contact`.
   - Explain that the CLI developer app/user authorization and the group-facing bot are two separate components. Do not assume that installing this skill creates or invites a bot.
   - Before creating anything, search for an existing target group, Base, tables, views, roles, and Workflow. If setup was interrupted, continue from the existing resources instead of creating duplicates.
   - Present missing prerequisites and browser/manual checkpoints plainly. Do not claim end-to-end readiness until the CLI, authorization, companion skills, and required manual bot steps have been verified.

1. Confirm scope with the user:
   - Existing group or create a new group?
   - Existing bot or add/invite one? The lab must provide or create the intelligent bot itself; this skill briefs the bot and wires permissions/workflow around it.
   - Should group members view only or edit the task tracker?
   - Who should retain edit access to the `Update Log` audit table? Default to the Base owner, user-selected maintainers, and the bot or service account; ordinary members should be read-only there.
   - Weekly reminder time and timezone.
   - The weekly data-maintenance cutoff. Recommend that members update tasks continuously and, at minimum, review current-stage deadlines before the scheduled Monday report.
   - Whether the weekly report should include the default three views: `This Week`, `Teacher Confirmation`, and `Recent 4 Weeks Completed`.
   - Whether one designated maintainer needs special audit attribution. If yes, resolve that person's display name and stable Feishu/Lark user ID at deployment time; otherwise attribute every human edit to the actual modifier. Never store deployment identities in reusable skill files.
   - Whether messages to the group must be previewed for user approval before sending.
   - Whether the group-facing bot may write routine progress itself. If yes, confirm the exact execution identity and the narrow runtime operations it may use; Base sharing alone is not a tool authorization policy.

2. Prepare Feishu/Lark CLI:
   - If `lark-cli` or the companion skills are missing, follow `references/getting-started.md`; do not improvise an installation command.
   - Run `lark-cli doctor`.
   - Prefer `--as user` for creating Base files, granting permissions, and sending messages as the user.
   - If user identity is missing, run device-flow auth. Show the authorization URL and QR code, then wait for the user to confirm before continuing.
   - If scopes are missing, request only the needed domains first: `base`, `drive`, `im`, plus `contact` when resolving member names, owners, or identity-based permissions.

3. Create or locate the group:
   - Search by group name with `lark-cli im +chat-search`.
   - If the user asks to create a group, use `lark-cli im +chat-create`.
   - Read the real `chat_id`; never guess it.
   - Reuse a matching existing group after confirming it with the user. Never create a second group merely because a prior run stopped midway.

4. Create the Base:
   - Search the user's drive for a matching Base before creation. Inspect its tables and schema; reuse and repair a partial setup when it is clearly the intended tracker.
   - Create a Base named like `<Lab/Project> Task Tracker`.
   - Create the first table named `Task Register`.
   - Use the schema in `references/base-template.md`.
   - Add an `Update Log` table and link it back to `Task Register`, so task rows show clickable update history while the main table stays focused on current state.
   - Add an internal `Status Snapshot` field to `Task Register` when automatic audit logging is enabled; hide it from routine views.
   - Add an internal `Update Source` text field when CLI/bot writes must be distinguished from manual edits; hide it from every routine view.
   - Add an internal text-returning formula field such as `Current Week Marker` when the tracker includes a current-week view. Use `TODAY()` and `WEEKDAY()` so the marker moves automatically each week; never freeze the view to setup-week dates.
   - Add a text-returning `Recent Completion Marker` formula when weekly reports need a rolling four-week completed view.
   - Add categories and views exactly enough for the lab to start; avoid overbuilding.

5. Configure permissions:
   - Grant the group chat `edit` permission when the lab members should update tasks themselves.
   - Set public link sharing conservatively, usually `tenant_readable`, with external sharing disabled.
   - If advanced Base permissions are available, allow ordinary members to edit `Task Register` but make `Update Log` read-only. Keep `Update Log` edit/write access only for the Base owner, user-selected maintainers, and bot or service account. If role-member assignment is not fully exposed by CLI, tell the user exactly what must be finished in the Feishu/Lark UI.
   - Resolve the real execution identity used by the group-facing bot. Do not assume that the CLI developer app, the bot displayed in the chat, and the account writing Base records are the same identity.
   - Verify three independent layers for bot writes: required app/API scopes, Base collaborator plus advanced-role access, and the bot runtime's tool allowlist. Passing one layer does not prove the other two are ready.
   - Report the permission model plainly to the user.

6. Configure views:
   - Create the views in `references/base-template.md`.
   - Add `Status-Sorted Table` / `总表（按状态排序）`. Keep the status option order active-first and `Completed` last, so completed work does not occupy the first screen as the tracker grows.
   - Name the owner-grouped accountability view `Owner Assignment` / `负责人任务分工`, not a generic member-workload name. Group it only by `Owner`; collaborators remain visible but do not become accountable owners.
   - Add `Member Participation` / `成员参与任务` as an ungrouped lookup view that displays both `Owner` and `Collaborators`. Let users search or filter this view by a member name; do not present it as exact per-member workload statistics.
   - If the user needs exact workload counts that include collaborators, ask before creating a normalized `Member Assignment` table with one row per task-member-role. Confirm how it will stay synchronized and explain any Workflow quota impact; do not infer independent member groups from a comma-separated collaborator field.
   - Implement `This Week` with the dynamic `Current Week Marker` formula and filter for the text result `This Week`. Do not use fixed `ExactDate(...)` boundaries for a reusable current-week view.
   - Define `This Week` strictly as records with an explicit deadline from Monday 00:00 inclusive to the next Monday 00:00 exclusive, with status not `Completed`. Do not infer weekly membership from `Latest Progress`, `Research Plan`, or phrases such as "finish this week".
   - Add `Recent 4 Weeks Completed` using the rolling marker; use it as the completed-work source for weekly summaries.
   - Hide `Current Week Marker` from every routine view after creating it. New Base fields may be added to existing views automatically; reapply each view's visible-field list and verify the kanban separately. If a kanban name-based update is a no-op, retry with real field IDs.
   - Use `Teacher Confirmation` to show records where the confirmation/callback field is not empty.
   - Use the status kanban for the overview; records that require teacher confirmation should usually have `Status = Teacher Confirmation`, so the kanban column is not empty.

7. Brief the bot:
   - Search group bots and get the real bot open_id.
   - If no bot exists, tell the user to create/invite one first. Options include Feishu/Lark Aily-style bots, a custom Feishu/Lark app bot, or an external agent such as Codex/Claude Code connected to Feishu/Lark through an approved bot/app integration.
   - If the bot is expected to update the Base itself, ensure its actual execution identity has message-read/message-send access, Base collaborator access, and an advanced Base role that can edit `Task Register` and append `Update Log`. If not, keep it as an assistant that proposes updates for a human to confirm.
   - Separately configure a least-privilege runtime tool allowlist for member-triggered progress updates. Allow reading/searching existing tasks; updating only `Status`, `Latest Progress`, and the internal `Update Source`; appending and linking an `Update Log` row; and syncing `Status Snapshot` plus clearing `Update Source`. Deny task creation/deletion, owner/deadline/teacher-confirmation changes, other fields, views, Workflow, and permissions unless a separately authorized workflow explicitly permits them.
   - Bind write approval to the exact preview in the same conversation or message thread. If the preview changes, ask for confirmation again. A generic confirmation from another context must not authorize the write.
   - Run one read test and one confirmed write-plus-log test through the group bot's real runtime before calling it ready. Document access and runtime tool access must both pass.
   - Draft the bot instruction message using `references/bot-brief.md`.
   - For a scheduled weekly report, read `references/weekly-automation.md`. Replace the whole scheduled prompt in one operation, use direct executable instructions and full CLI flags, and never send a partial prompt assembled across multiple chat messages.
   - If the user has requested approval-before-send, show the draft in chat and wait for confirmation before sending.
   - Use a real mention when possible: `<at id=BOT_OPEN_ID></at>`.

8. Add initial tasks:
   - Convert free-form lab instructions into rows with fields from the Base schema.
   - Put plans/designs in `Research Plan`, not `Latest Progress`.
   - Leave `Latest Progress` empty until the work actually starts or changes.
   - Convert relative deadlines into absolute dates. For "this week" or "next week", include the Sunday date in display text and write the date field as `YYYY-MM-DD 23:59:00`.
   - For long-running experiments, treat `Deadline` as the current stage's committed completion date, not necessarily the predicted end date of the entire experiment. When useful, split a long experiment into a dated stage task or next action. A task without an explicit current-week deadline remains in the main tracker but is excluded from `This Week`.
   - Always preview additions or changes as a Markdown table before touching the Base.
   - If the user edits the draft, render the revised table again. Repeat until the user explicitly confirms upload/write.
   - Do not create, update, delete, or change task status in the Base from a draft alone. Wait for an explicit confirmation such as "upload", "write it", "push to the board", or equivalent.

9. Archive task updates:
   - Treat the task row as the current final state only.
   - For every confirmed task update, update the task row to the latest state and append a linked record in `Update Log`.
   - Link the update record to the task through the bidirectional `Update Log` / `Task` relation.
   - Use the update record to preserve process details, previous status, new status, submitter, and context that should not clutter the main task row.
   - When the team will edit `Task Register` directly, configure the source-aware Base Workflow in `references/audit-workflow.md`: append a linked row to `Update Log`, attribute it to CLI/bot or the actual human editor, then sync `Status Snapshot` and clear `Update Source`.
   - For every CLI/bot task write, set `Update Source` to the configured machine-source label in the same patch. Human edits must not set that field.
   - Give every attribution branch its own cleanup action. A typical setup has machine and human branches; add an optional designated-maintainer branch only when the user wants special attribution. Nested Feishu/Lark branches may discard a shared cleanup node even when an update response echoes the requested links.
   - Display `Feishu CLI` / `Lark CLI` for CLI writes and the actual modifier for human edits. When the user explicitly configures a designated maintainer branch, use that maintainer's deployment-time display name. Do not expose numeric account aliases or internal IDs in user-facing log fields.
   - If bot/API writes do not trigger the Base Workflow in that tenant, the bot/API update must create the `Update Log` row in the same confirmed operation.

10. Validate:
   - Read back Base fields, views, permissions, and sample records.
   - Verify that `Owner Assignment` is grouped only by owner and that `Member Participation` visibly includes both owners and collaborators. Do not claim collaborator-inclusive workload counts unless a normalized assignment table exists.
   - Read records through `This Week`: the current-week view must not retain setup-week records.
   - Independently validate every `This Week` record against the current timezone's Monday-to-next-Monday date range and `Status != Completed`; do not trust the view name alone.
   - Read `Recent 4 Weeks Completed` and verify it rolls with `TODAY()` rather than setup-time dates.
   - After every Workflow update, independently call workflow-get. Verify all three source branches retain their cleanup links and the workflow remains enabled; do not trust only the update response.
   - Scan all skill files before sharing. Reject real names, usernames, open IDs, Base/table/view/workflow tokens, unpublished identifiers, and institution-specific details.
   - Summarize what was created and provide the Base URL.

## Update Rules

Use these default operating rules unless the user says otherwise:

- Prefer natural-language, draft-first updates in Codex followed by confirmed batch writes through `lark-cli`, especially for multiple changes.
- The bot can capture fragment notes privately and extract proposed tasks or progress from group chat.
- Members may also update routine progress directly in the Base or mention the bot in the group.
- For a group mention that reports progress on one existing task, first match exactly one existing record. If multiple records match, ask only which task was meant; do not ask experimental follow-up questions.
- Return a minimal preview containing only task name, proposed status, and proposed latest progress. Rewrite only facts the member stated. Do not infer follow-up work, sample counts, interpretations, blockers, or questions.
- Map explicit `completed` language to `Completed`; map `in progress`, `submitted`, `results returned`, or `being analyzed` to `In Progress`; otherwise preserve the existing status. The upload confirmation is also the required confirmation for a completion mark.
- Do not create, clear, or modify `Teacher Confirmation` unless the member explicitly mentions a teacher/supervisor decision or supplies the confirmation question. Silence about the teacher means leave the field unchanged.
- After the preview, ask only whether to upload. On explicit confirmation, update only the approved fields, set the machine update source, and append the linked update-log row. If writing tools or permissions are unavailable, say that no write occurred and name the missing access; never imply success.
- The bot should ask for a separate expanded confirmation before creating new tasks, changing owner/deadline, deleting records, or changing a task to/from teacher-confirmation status.
- An authorized member confirms ambiguous instructions by restating task goal, owner, deadline, deliverable, and confirmation points.
- For agent-assisted task entry, use a strict draft-first loop: draft table -> user edits -> revised table -> explicit upload confirmation -> Base write. Never skip the preview table.
- For confirmed updates, keep `Task Register` as the current state and append every change to `Update Log`; do not use the main task row as the long-form history archive.
- Treat `Update Log` as an audit table. Do not ask ordinary members to edit it directly; protect it with read-only permissions where possible. Only the Base owner, user-selected maintainers, and the bot/service account should write or repair log rows.
- If direct edits to `Task Register` are allowed, enable automatic audit logging through Base Workflow; otherwise ensure every bot or CLI task update also appends a linked `Update Log` record.
- Keep audit attribution source-aware: machine writes use a generic CLI/bot label; human writes use the person's display name. Never publish account aliases or stable IDs in reusable templates.
- Every configured audit branch must finish by syncing `Status Snapshot` and clearing `Update Source`, including CLI writes, ordinary human edits, and any optional designated-maintainer branch.
- When a task is marked `Completed`, clear the teacher-confirmation/blocker field by default so completed work does not remain in the confirmation view. Only keep or write blocker text for completed tasks when the user explicitly asks for it.
- Ask members to maintain progress and current-stage deadlines during normal work. At minimum, run a deadline review before the scheduled weekly report; tasks intended for completion that week should receive an explicit date in that week.
- The weekly report's `This Week` section contains only non-completed tasks with explicit deadlines in the current Monday-through-Sunday window. Never substitute all open or in-progress tasks, and never infer dates from free text.
- If the report prints a week number, calculate the ISO-8601 week from the report date in the configured timezone. Do not reuse the previous reporting period's week number.

## References

Read only the needed reference file:

- `references/getting-started.md`: first-use installation, authentication, component boundaries, preflight, resume behavior, and troubleshooting.
- `references/base-template.md`: Base table fields, category options, view order, and example anonymous rows.
- `references/cli-commands.md`: lark-cli command patterns with placeholders.
- `references/bot-brief.md`: reusable bot instruction message and weekly summary format.
- `references/audit-workflow.md`: source-aware audit Workflow topology, attribution rules, cleanup branches, and readback validation.
- `references/weekly-automation.md`: robust scheduled weekly-report prompt, three-view data contract, failure handling, and duplicate-send prevention.
