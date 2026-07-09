---
name: lark-lab-task-group
description: "Build a Feishu/Lark research-lab task group from zero to one using lark-cli: create or locate a group chat, add/brief a bot, create a Base task tracker, configure fields/categories/views/permissions, post the tracker to the group, and set lightweight rules for task intake, progress updates, teacher confirmation, weekly reminders, and summaries. Use when researchers want a reusable lab task-management template in Feishu/Lark, especially for student-supervisor groups, wet-lab workflows, sample submission tracking, planting/experiment schedules, and multi-person progress coordination."
---

# Lark Lab Task Group

## Purpose

Set up a lightweight lab task workflow in Feishu/Lark:

- Group chat for discussion.
- Base task tracker for status and accountability.
- Bot for organizing chat content, reminding people, and producing weekly summaries.
- Clear rules for when people update the Base directly versus when the bot asks for confirmation.

Always protect privacy. Do not include real gene names, sample names, unpublished project names, people's names, or institution-specific details in reusable templates. Use neutral examples such as `Genotype A`, `Material group`, `Student A`, and `Assistant`.

## Workflow

1. Confirm scope with the user:
   - Existing group or create a new group?
   - Existing bot or add/invite one? The lab must provide or create the intelligent bot itself; this skill briefs the bot and wires permissions/workflow around it.
   - Should group members view only or edit the task tracker?
   - Weekly reminder time and timezone.
   - Whether messages to the group must be previewed for user approval before sending.

2. Prepare Feishu/Lark CLI:
   - Run `lark-cli doctor`.
   - Prefer `--as user` for creating Base files, granting permissions, and sending messages as the user.
   - If user identity is missing, run device-flow auth. Show the authorization URL and QR code, then wait for the user to confirm before continuing.
   - If scopes are missing, request only the needed domains first: `base`, `drive`, `im`, and optionally `contact`.

3. Create or locate the group:
   - Search by group name with `lark-cli im +chat-search`.
   - If the user asks to create a group, use `lark-cli im +chat-create`.
   - Read the real `chat_id`; never guess it.

4. Create the Base:
   - Create a Base named like `<Lab/Project> Task Tracker`.
   - Create the first table named `Task Register`.
   - Use the schema in `references/base-template.md`.
   - Add an `Update Log` table and link it back to `Task Register`, so task rows show clickable update history while the main table stays focused on current state.
   - Add categories and views exactly enough for the lab to start; avoid overbuilding.

5. Configure permissions:
   - Grant the group chat `edit` permission when the lab members should update tasks themselves.
   - Set public link sharing conservatively, usually `tenant_readable`, with external sharing disabled.
   - Report the permission model plainly to the user.

6. Configure views:
   - Create the views in `references/base-template.md`.
   - Use `Teacher Confirmation` to show records where the confirmation/callback field is not empty.
   - Use the status kanban for the overview; records that require teacher confirmation should usually have `Status = Teacher Confirmation`, so the kanban column is not empty.

7. Brief the bot:
   - Search group bots and get the real bot open_id.
   - If no bot exists, tell the user to create/invite one first. Options include Feishu/Lark Aily-style bots, a custom Feishu/Lark app bot, or an external agent such as Codex/Claude Code connected to Feishu/Lark through an approved bot/app integration.
   - If the bot is expected to update the Base itself, ensure it has message-read/message-send access, Base access, and permission to edit the tracker. If not, keep it as an assistant that proposes updates for a human to confirm.
   - Draft the bot instruction message using `references/bot-brief.md`.
   - If the user has requested approval-before-send, show the draft in chat and wait for confirmation before sending.
   - Use a real mention when possible: `<at id=BOT_OPEN_ID></at>`.

8. Add initial tasks:
   - Convert free-form lab instructions into rows with fields from the Base schema.
   - Put plans/designs in `Research Plan`, not `Latest Progress`.
   - Leave `Latest Progress` empty until the work actually starts or changes.
   - Convert relative deadlines into absolute dates. For "this week" or "next week", include the Sunday date in display text and write the date field as `YYYY-MM-DD 23:59:00`.
   - Always preview additions or changes as a Markdown table before touching the Base.
   - If the user edits the draft, render the revised table again. Repeat until the user explicitly confirms upload/write.
   - Do not create, update, delete, or change task status in the Base from a draft alone. Wait for an explicit confirmation such as "upload", "write it", "push to the board", or equivalent.

9. Archive task updates:
   - Treat the task row as the current final state only.
   - For every confirmed task update, update the task row to the latest state and append a linked record in `Update Log`.
   - Link the update record to the task through the bidirectional `Update Log` / `Task` relation.
   - Use the update record to preserve process details, previous status, new status, submitter, and context that should not clutter the main task row.

10. Validate:
   - Read back Base fields, views, permissions, and sample records.
   - Summarize what was created and provide the Base URL.

## Update Rules

Use these default operating rules unless the user says otherwise:

- Group members update routine progress directly in the Base.
- The bot can extract proposed tasks and progress from chat.
- The bot should ask for confirmation before creating new tasks, changing owner/deadline, marking complete, deleting records, or changing a task to/from teacher-confirmation status.
- The student coordinator or user confirms ambiguous teacher instructions by restating task goal, owner, deadline, deliverable, and confirmation points.
- For agent-assisted task entry, use a strict draft-first loop: draft table -> user edits -> revised table -> explicit upload confirmation -> Base write. Never skip the preview table.
- For confirmed updates, keep `Task Register` as the current state and append every change to `Update Log`; do not use the main task row as the long-form history archive.
- When a task is marked `Completed`, clear the teacher-confirmation/blocker field by default so completed work does not remain in the confirmation view. Only keep or write blocker text for completed tasks when the user explicitly asks for it.

## References

Read only the needed reference file:

- `references/base-template.md`: Base table fields, category options, view order, and example anonymous rows.
- `references/cli-commands.md`: lark-cli command patterns with placeholders.
- `references/bot-brief.md`: reusable bot instruction message and weekly summary format.
