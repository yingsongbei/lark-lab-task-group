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
13. `Created Time` - created_at.
14. `Updated Time` - updated_at.
15. `Update Log` - link to `Update Log`; created automatically when the update-log table uses a bidirectional link field.

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
- `Biochemical Submission`
- `Biochemistry`

For Chinese teams, use:

- `文献`
- `实验`
- `数据`
- `写作`
- `会议`
- `杂事`
- `种苗（水培）`
- `生化送测`
- `生化`

`Status`:

- `Not Started` / `未开始`
- `In Progress` / `进行中`
- `Teacher Confirmation` / `待老师确认`
- `Completed` / `已完成`
- `Paused` / `暂缓`

`Priority`:

- `High` / `高`
- `Medium` / `中`
- `Low` / `低`

## Views

Create these views:

1. `Task Entry` / `任务录入表`
   - Type: grid.
   - Visible field order:
     `Task Name`, `Task Category`, `Owner`, `Collaborators`, `Deadline`, `Status`, `Latest Progress`, `Update Log`, `Teacher Confirmation / Blocker`, `Priority`, `Deliverable`, `Research Plan`, `Related Materials`, `Created Time`, `Updated Time`.

2. `Overview Kanban` / `总览看板`
   - Type: kanban.
   - Group by `Status`.
   - If a record has teacher-confirmation content, set `Status` to `Teacher Confirmation` so this column is populated.

3. `This Week` / `本周任务`
   - Type: grid.
   - Filter deadline from Monday 00:00 through Sunday 23:59 for the current week.
   - Sort by `Deadline` ascending.

4. `Teacher Confirmation` / `待老师确认`
   - Type: grid.
   - Filter: `Teacher Confirmation / Blocker` is not empty.
   - Visible field order:
     `Task Name`, `Teacher Confirmation / Blocker`, `Owner`, `Collaborators`, `Deliverable`, `Research Plan`, `Update Log`, `Deadline`, `Status`, `Task Category`.

5. `Member Workload` / `成员任务分工`
   - Type: grid.
   - Group by `Owner`.
   - Sort by `Deadline` ascending.

6. `Update Log Table` / `更新记录表`
   - Table: `Update Log`.
   - Type: grid.
   - Sort by `Submitted Time` / `提交时间` descending.
   - Visible field order:
     `Update Title`, `Task`, `Update Type`, `Update Content`, `Previous Status`, `New Status`, `Submitted By`, `Notes`, `Submitted Time`.

Do not create a personal `My Tasks` view by default unless the user asks for it.

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
