# Base Template

Use this as the default research-lab task tracker structure.

## Base And Table

- Base name: `<Project or Lab> Task Tracker`
- Table name: `Task Register`
- History table name: `Update Log`

## Fields

Create these fields in this order:

1. `Task Name` - text; primary field.
2. `Task Category` - select.
3. `Owner` - text.
4. `Collaborators` - text.
5. `Deadline` - datetime, format `yyyy-MM-dd HH:mm`.
6. `Status` - select.
7. `Latest Progress` - text.
8. `Teacher Confirmation / Blocker` - text.
9. `Priority` - select.
10. `Deliverable` - text.
11. `Research Plan` - text.
12. `Related Materials` - text.
13. `Update Log` - link to `Update Log`; created automatically when the update-log table uses a bidirectional link field.
14. `Created Time` - created_at.
15. `Updated Time` - updated_at.
16. `Status Snapshot` - text; internal field for audit workflow only. Hide it from routine views.
17. `Update Source` - text; internal machine-write marker for source-aware audit logging. Hide it from routine views.
18. `Current Week Marker` - formula returning text; internal field for the dynamic current-week view. Hide it from routine views.
19. `Recent Completion Marker` - formula returning text; internal field for the rolling four-week completed view. Hide it from routine views.

Recommended English formula:

```text
IF(ISBLANK([Deadline]), "Not This Week", IF(AND([Deadline] >= TODAY() - WEEKDAY(TODAY(), 2) + 1, [Deadline] < TODAY() - WEEKDAY(TODAY(), 2) + 8), "This Week", "Not This Week"))
```

For a Chinese tracker, use field `本周标记`, deadline field `截止时间`, and results `本周` / `非本周`.

For research projects, interpret `Deadline` as the committed date for the task's current stage or next deliverable. It does not have to predict the end of the entire experiment. Leave it blank when no stage commitment exists, but understand that the record will not appear in `This Week`. For clearer accountability, split a long experiment into dated stage tasks.

Recommended rolling completion formula:

```text
IF(AND([Status] = "Completed", NOT(ISBLANK([Updated Time])), [Updated Time] >= TODAY() - 28), "Recent 4 Weeks Completed", "Other")
```

For a Chinese tracker, use fields `状态` and `更新时间`, and results `近四周已完成` / `其他`. This view treats the task's latest update time as the completion activity time; teams that need a legally strict completion timestamp should add a dedicated `Completed Time` field instead.

## Update Log Table

Create a second table named `Update Log`. Use these fields:

1. `Update Title` - text; primary field.
2. `Task` - link to `Task Register`; bidirectional, with the reverse field named `Update Log`.
3. `Update Type` - select.
4. `Update Content` - text.
5. `Previous Status` - text.
6. `New Status` - text.
7. `Submitted By` - text.
8. `Notes` - text.
9. `Submitted Time` - created_at, format `yyyy-MM-dd HH:mm`.

Chinese field names:

1. `提交标题`
2. `关联任务`
3. `提交类型`
4. `提交内容`
5. `更新前状态`
6. `更新后状态`
7. `提交人`
8. `备注`
9. `提交时间`

`Update Type` / `提交类型` options:

- `New Task` / `新建任务`
- `Progress Update` / `进展更新`
- `Status Change` / `状态变更`
- `Deadline Change` / `截止时间变更`
- `Owner/Collaborator Change` / `负责人/协作人变更`
- `Teacher Confirmation / Blocker` / `老师确认/卡点`
- `Other` / `其他`

## Select Options

`Task Category`:

- `Literature`
- `Experiment`
- `Data`
- `Writing`
- `Meeting`
- `Misc`
- `Seedling / Hydroponics`
- `Seedling / Soil`
- `Biochemical Submission`
- `Biochemistry`
- `System Setup`
- `Identification`
- `Mailing`
- `Learning`

For Chinese teams, use:

- `文献`
- `实验`
- `数据`
- `写作`
- `会议`
- `杂事`
- `种苗（水培）`
- `种苗（土培）`
- `生化送测`
- `生化`
- `体系搭建`
- `鉴定`
- `邮寄`
- `学习`

`Status`:

- `Not Started` / `未开始`
- `In Progress` / `进行中`
- `Teacher Confirmation` / `待老师确认`
- `Paused` / `暂缓`
- `Completed` / `已完成`

`Priority`:

- `High` / `高`
- `Medium` / `中`
- `Low` / `低`

## Views

Create these views:

1. `Task Entry` / `任务录入表`
   - Type: grid.
   - Visible field order:
     `Task Name`, `Task Category`, `Owner`, `Collaborators`, `Deadline`, `Status`, `Latest Progress`, `Teacher Confirmation / Blocker`, `Priority`, `Deliverable`, `Research Plan`, `Related Materials`, `Update Log`, `Created Time`, `Updated Time`.

2. `Status-Sorted Table` / `总表（按状态排序）`
   - Type: grid.
   - Sort by `Status` ascending, `Deadline` ascending, then `Updated Time` descending.
   - Keep the status-option order `Not Started`, `In Progress`, `Teacher Confirmation`, `Paused`, `Completed`, so completed work appears last.

3. `Overview Kanban` / `总览看板`
   - Type: kanban.
   - Group by `Status`.
   - Keep the `Completed` group last.
   - If a record has teacher-confirmation content, set `Status` to `Teacher Confirmation` so this column is populated.

4. `This Week` / `本周任务`
   - Type: grid.
   - Filter `Current Week Marker = This Week` / `本周标记 = 本周` and `Status != Completed` / `状态 != 已完成`.
   - The marker formula must calculate Monday 00:00 through the next Monday 00:00 from `TODAY()`; do not store setup-week dates in the view filter.
   - A blank deadline is not a current-week task. Do not infer weekly membership from progress or plan text.
   - Sort by `Deadline` ascending.

5. `Teacher Confirmation` / `待老师确认`
   - Type: grid.
   - Filter: `Teacher Confirmation / Blocker` is not empty.
   - Visible field order:
     `Task Name`, `Teacher Confirmation / Blocker`, `Owner`, `Collaborators`, `Deliverable`, `Research Plan`, `Update Log`, `Deadline`, `Status`, `Task Category`.

6. `Owner Assignment` / `负责人任务分工`
   - Type: grid.
   - Group by `Owner`.
   - Sort by `Deadline` ascending.
   - Treat this as the accountability view: one task has one accountable owner. Keep `Collaborators` visible, but do not imply that collaborator-only members are missing from the tracker.

7. `Member Participation` / `成员参与任务`
   - Type: grid.
   - Do not group by `Owner` or by a comma-separated `Collaborators` text value.
   - Visible field order:
     `Task Name`, `Owner`, `Collaborators`, `Status`, `Deadline`, `Latest Progress`, `Teacher Confirmation / Blocker`, `Priority`, `Task Category`, `Deliverable`, `Research Plan`, `Related Materials`, `Update Log`, `Created Time`, `Updated Time`.
   - Sort by `Status` ascending, `Deadline` ascending, then `Updated Time` descending.
   - Use member-name search or a temporary filter to inspect everything a person participates in. This is a participation lookup, not an exact per-member workload count.

8. `Recent 4 Weeks Completed` / `近四周已完成`
   - Type: grid.
   - Filter `Recent Completion Marker = Recent 4 Weeks Completed` / `近四周完成标记 = 近四周已完成`.
   - Sort by `Updated Time` descending.
   - Use this view as the completed-work input for the weekly report.

9. `Update Log Table` / `更新记录表`
   - Table: `Update Log`.
   - Type: grid.
   - Sort by `Submitted Time` / `提交时间` descending.
   - Visible field order:
     `Update Title`, `Task`, `Update Type`, `Update Content`, `Previous Status`, `New Status`, `Submitted By`, `Notes`, `Submitted Time`.

Do not create a personal `My Tasks` view by default unless the user asks for it.

### Owner And Participation Views

- Keep ownership and participation separate. `Owner Assignment` answers who is accountable; `Member Participation` answers where a person is involved.
- A task with multiple collaborators cannot be counted once under every person by grouping the existing `Owner` or comma-separated `Collaborators` text fields. Never label such a grouping as exact member workload statistics.
- When exact collaborator-inclusive counts are required, offer an optional normalized `Member Assignment` table with fields such as `Member`, linked `Task`, and `Role` (`Owner` or `Collaborator`), plus status/deadline lookups. Store one row per task-member-role.
- Before adding that table, confirm whether the bot/CLI, a Workflow, or a human maintainer will synchronize it. Explain that a Workflow-based sync can consume additional monthly runs. Keep this advanced table out of the default lightweight setup unless the user requests exact counts.

### Current-Week View Bug Prevention

- Never implement a reusable `This Week` view with fixed `ExactDate(...)` values. That only shows the week in which the template was created.
- Make the formula return text instead of Boolean. The Base view-filter API accepts a single string or number for formula fields, but may reject a Boolean filter value.
- After creating the internal marker, explicitly restore visible fields for all routine views because Base may auto-add new fields to existing views.
- Verify kanban visibility separately. If setting kanban visible fields by name produces a no-op, submit the same list with real field IDs.
- Apply the same hide-and-verify pass to `Update Source`, `Status Snapshot`, and `Recent Completion Marker`.

### Completed-Last View Rule

- Keep an all-task grid sorted by status, deadline, and latest update so active work stays at the top and `Completed` remains last.
- Do not rely on creation order. Confirm the status-option order or use a dedicated sort key when the tenant does not respect select-option order.
- Keep completed tasks accessible through `Recent 4 Weeks Completed` and the overview kanban instead of allowing them to dominate the first screen.

## Permission Model

Ask the user who should keep edit access to the audit table before configuring advanced permissions.

Default model:

1. `Task Register`: ordinary group members may edit if the lab wants direct progress updates.
2. `Update Log`: ordinary group members are read-only.
3. `Update Log` write access: keep only for the Base owner, user-selected maintainers, and the bot or service account that writes audit rows.
4. Public link sharing: use tenant-only access by default and disable external sharing unless the user explicitly asks otherwise.

For a group-facing bot, resolve the identity that actually writes Base records. The chat bot, CLI developer app, and runtime service account may be different identities. Grant that execution identity collaborator access and, when advanced permissions are enabled, a role that can edit `Task Register` and append `Update Log`.

Document access is only one permission layer. Separately configure the bot runtime's least-privilege tool allowlist. For ordinary member-triggered progress reports, allow task lookup; confirmed changes to `Status`, `Latest Progress`, and internal source/snapshot fields; and append-only linked audit writes. Deny task creation/deletion, owner/deadline/teacher-confirmation changes, schema/views, Workflow, and permission administration unless a separate authorized workflow explicitly allows them.

If the CLI cannot assign every role member in the current tenant, create the roles and table permissions with CLI where possible, then clearly tell the user which people or bots must be assigned in the Feishu/Lark UI. Read one known task and perform one confirmed progress-write-plus-log test through the real group bot runtime before reporting that bot editing is ready.

## Automatic Audit Workflow

When members are allowed to edit `Task Register` directly, configure a Base Workflow so manual changes still create update history.

Use the source-aware topology in `audit-workflow.md`. It distinguishes CLI/bot writes from human edits and optionally recognizes a user-selected designated maintainer. Every configured branch must use its own cleanup action to sync `Status Snapshot` and clear `Update Source`; independently read the saved workflow back because nested branches may silently discard a shared cleanup node.

Hide `Status Snapshot` and `Update Source` from `Task Entry`, `Status-Sorted Table`, `Overview Kanban`, `Teacher Confirmation`, `Owner Assignment`, `Member Participation`, and every other routine view.

Important fallback: if bot/API/CLI writes do not trigger Workflow in the user's tenant, the bot/API/CLI path must create the linked `Update Log` row, sync the snapshot, and clear the source marker in the same confirmed operation.

## Anonymous Example Rows

Use examples like these when demonstrating the template. Never include real private names or unpublished biological identifiers.

| Task Name | Task Category | Owner | Collaborators | Deadline | Status | Deliverable | Research Plan | Latest Progress | Teacher Confirmation / Blocker |
|---|---|---|---|---|---|---|---|---|---|
| Plant genotype and material seedlings for this week | Seedling / Hydroponics | Student A | Assistant | This week (Sunday date) | Not Started | Seedlings planted and layout recorded | Plant genotype A and material group B in separate boxes; use interval planting for sampling and separated rows for uptake-rate measurement. |  |  |
| Submit material group for IP-MS | Biochemical Submission | Student B | Student A | TBD | Teacher Confirmation | IP-MS samples with three biological replicates | Start with three anonymized genotypes; decide whether to submit whole-protein MS after reviewing IP-MS results. |  | Confirm sample count, treatment groups, and replicate number. |
| Submit material group for free amino acid assay | Biochemical Submission | Student B | Student A | Next week (Sunday date) | Teacher Confirmation | Free amino acid assay samples | Use the same material batch if appropriate; align sampling stage and storage conditions. |  | Confirm genotype/treatment/replicate matrix. |
| Decide whether to run whole-protein MS | Biochemistry | Student B | Student A | After IP-MS results | Paused | Decision note | Wait for IP-MS results before making the next submission decision. |  |  |
| Submit single-cell sequencing samples | Biochemical Submission | Student B | Student A | TBD | Teacher Confirmation | Sequencing submission samples | Use the newly planted seedlings; coordinate with sequencing vendor. |  | Confirm sample number, tissue type, and replicate design. |
| Subcellular fluorescence observation | Biochemistry | Student B | Student A | TBD | Not Started | Fluorescence images or observation record | Use the same seedling batch; arrange microscope access at an appropriate platform. |  |  |

## Draft-First Entry Protocol

Before creating or updating records in the Base, always render proposed rows as a Markdown table with the task fields. Treat this table as a draft.

Do not write to the Base until the user explicitly confirms upload/write. If the user modifies the draft, render the revised table again and wait for confirmation.

## Update Archive Protocol

Use `Task Register` as the current-state table only. After a confirmed update:

1. Modify the current fields in `Task Register`.
2. Add one row to `Update Log`.
3. Link that `Update Log` row to the task.

When a task status becomes `Completed` / `已完成`, clear `Teacher Confirmation / Blocker` / `卡点/需老师确认` unless the user explicitly asks to keep or add blocker text. This keeps completed tasks out of the teacher-confirmation view.

This keeps the main task list readable while preserving a clickable history trail.

For direct manual edits in `Task Register`, prefer the automatic audit workflow above so the log is not dependent on everyone remembering to write a second row manually.
