# lark-cli Command Patterns

Replace placeholders such as `BASE_TOKEN`, `TABLE_ID`, `VIEW_ID`, `CHAT_ID`, and `BOT_OPEN_ID`. Always read real IDs from command output.

## Health And Auth

```powershell
lark-cli doctor
lark-cli auth status
lark-cli auth login --domain base --domain drive --domain im --no-wait --json
lark-cli auth qrcode "<verification_url>" --output ".tmp\lark-auth.png"
lark-cli auth login --device-code "<device_code>"
```

If creating a new CLI app for a new tenant:

```powershell
lark-cli config init --new --force-init --brand feishu --lang zh_cn
```

## Search Group And Bot

```powershell
lark-cli im +chat-search --as user --query "<group name>" --chat-modes group --format json
lark-cli im chat.members bots --as user --chat-id CHAT_ID --format json
```

If no bot is returned, ask the user to create or invite a group bot first. This skill does not create a reasoning-capable bot by itself. Common choices are a Feishu/Lark Aily-style bot, a custom Feishu/Lark app bot, or an external agent connected through a compliant bot/app integration.

## Create Base

Use a JSON file for complex field definitions to avoid shell escaping issues.

```powershell
lark-cli base +base-create --as user `
  --name "<Project> Task Tracker" `
  --table-name "Task Register" `
  --fields "@fields.json" `
  --time-zone Asia/Shanghai `
  --format json
```

## Create Update Log Table

Create a second table for historical submissions. Use a bidirectional link so the main task table gets a clickable `Update Log` field.

```json
[
  { "name": "提交标题", "type": "text" },
  {
    "name": "关联任务",
    "type": "link",
    "link_table": "TASK_TABLE_ID",
    "bidirectional": true,
    "bidirectional_link_field_name": "更新记录"
  },
  {
    "name": "提交类型",
    "type": "select",
    "multiple": false,
    "options": [
      { "name": "新建任务" },
      { "name": "进展更新" },
      { "name": "状态变更" },
      { "name": "截止时间变更" },
      { "name": "负责人/协作人变更" },
      { "name": "老师确认/卡点" },
      { "name": "其他" }
    ]
  },
  { "name": "提交内容", "type": "text" },
  { "name": "更新前状态", "type": "text" },
  { "name": "更新后状态", "type": "text" },
  { "name": "提交人", "type": "text" },
  { "name": "备注", "type": "text" },
  { "name": "提交时间", "type": "created_at", "style": { "format": "yyyy-MM-dd HH:mm" } }
]
```

```powershell
lark-cli base +table-create --as user --base-token BASE_TOKEN `
  --name "更新记录" `
  --fields "@update-log-fields.json" `
  --format json
```

## Add Audit Snapshot Field

If direct edits to the task table should automatically create update-history rows, add an internal snapshot field to the task table. Hide this field from routine views.

```powershell
lark-cli base +field-create --as user --base-token BASE_TOKEN --table-id TASK_TABLE_ID `
  --json '{"name":"状态快照","type":"text"}' `
  --format json
```

Use `Status Snapshot` instead of `状态快照` for English trackers.

## Create Views

### Dynamic current-week and overdue views

Do not use fixed setup-week dates for a reusable `This Week` view. Create an internal formula that returns text; Boolean formula values may be rejected by the view-filter API.

```json
{
  "name": "Current Week Marker",
  "type": "formula",
  "description": "Internal field used by the dynamic current-week view.",
  "expression": "IF(ISBLANK([Deadline]), \"Not This Week\", IF(AND([Deadline] >= TODAY() - WEEKDAY(TODAY(), 2) + 1, [Deadline] < TODAY() - WEEKDAY(TODAY(), 2) + 8), \"This Week\", \"Not This Week\"))"
}
```

```powershell
lark-cli base +field-create --as user --base-token BASE_TOKEN --table-id TABLE_ID `
  --json "@current-week-marker.json" --i-have-read-guide --format json
```

For a Chinese tracker, use `本周标记`, `[截止时间]`, `本周`, and `非本周`.

Current-week filter:

```json
{"logic":"and","conditions":[["Current Week Marker","==","This Week"]]}
```

Overdue-unfinished filter:

```json
{"logic":"and","conditions":[["Deadline","<","Today"],["Status","disjoint",["Completed"]]]}
```

After creating the formula field, reapply visible-field lists to all routine views so the internal marker stays hidden. Verify kanban separately; when a name-based visible-field update returns a no-op, retry with real field IDs from `+field-list`.

```powershell
lark-cli base +view-rename --as user --base-token BASE_TOKEN --table-id TABLE_ID --view-id DEFAULT_VIEW_ID --name "Task Entry"

lark-cli base +view-create --as user --base-token BASE_TOKEN --table-id TABLE_ID `
  --json "@views.json" --format json
```

Set visible fields:

```powershell
lark-cli base +view-set-visible-fields --as user --base-token BASE_TOKEN --table-id TABLE_ID --view-id VIEW_ID `
  --json "@visible-fields.json" --format json
```

Set filters:

```powershell
lark-cli base +view-set-filter --as user --base-token BASE_TOKEN --table-id TABLE_ID --view-id VIEW_ID `
  --json "@filter.json" --format json
```

For "Teacher Confirmation", use:

```json
{
  "logic": "and",
  "conditions": [
    ["Teacher Confirmation / Blocker", "non_empty"]
  ]
}
```

Set grouping:

```powershell
lark-cli base +view-set-group --as user --base-token BASE_TOKEN --table-id TABLE_ID --view-id VIEW_ID `
  --json "{\"group_config\":[{\"field\":\"Status\",\"desc\":false}]}" --format json
```

## Grant Group Permission

Grant group edit permission when members should update the tracker:

```json
{
  "member_type": "openchat",
  "member_id": "CHAT_ID",
  "type": "chat",
  "perm": "edit"
}
```

```powershell
lark-cli drive permission.members create --as user --token BASE_TOKEN --type bitable `
  --data "@chat-edit-permission.json" --need-notification --yes --format json
```

Conservative public permission:

```json
{
  "link_share_entity": "tenant_readable",
  "share_entity": "same_tenant",
  "comment_entity": "anyone_can_view",
  "security_entity": "anyone_can_view",
  "external_access": false,
  "invite_external": false
}
```

```powershell
lark-cli drive permission.public patch --as user --token BASE_TOKEN --type bitable `
  --data "@public-permission.json" --yes --format json
```

## Protect Update Log With Advanced Permissions

Before configuring advanced permissions, ask the user which people and bots should retain edit access to `Update Log` / `更新记录`.

Recommended default:

- `Task Register` / `任务总表`: group members can edit if direct progress updates are expected.
- `Update Log` / `更新记录`: ordinary members are read-only.
- Update-log writers: Base owner, student/lab coordinator, named maintainers, and the bot or service account.

Enable advanced permissions:

```powershell
lark-cli base +advperm-enable --as user --base-token BASE_TOKEN --format json
```

List and inspect roles before changing them:

```powershell
lark-cli base +role-list --as user --base-token BASE_TOKEN --format json
lark-cli base +role-get --as user --base-token BASE_TOKEN --role-id ROLE_ID --format json
```

For an ordinary-member role, allow task-table editing but make the update-log table read-only:

```json
{
  "role_name": "Lab Member",
  "role_type": "custom_role",
  "table_rule_map": {
    "任务总表": { "perm": "edit" },
    "更新记录": { "perm": "read_only" }
  }
}
```

```powershell
lark-cli base +role-update --as user --base-token BASE_TOKEN --role-id MEMBER_ROLE_ID `
  --json "@member-role-update.json" --yes --format json
```

For a coordinator or bot/service-account role, keep both tables editable:

```json
{
  "role_name": "Task Maintainer",
  "role_type": "custom_role",
  "base_rule_map": { "copy": false, "download": false },
  "table_rule_map": {
    "任务总表": { "perm": "edit" },
    "更新记录": { "perm": "edit" }
  }
}
```

```powershell
lark-cli base +role-create --as user --base-token BASE_TOKEN `
  --json "@maintainer-role.json" --format json
```

If the current CLI or tenant cannot assign role members, finish membership assignment in the Feishu/Lark UI and report exactly which users/bots should be placed in each role.

## Create Automatic Update-Log Workflow

Use this when ordinary members may directly edit `任务总表`. First read real field IDs with `+field-list`; Workflow `ref` paths use field IDs such as `$.task_changed.fldXXXX`, not field names.

```powershell
lark-cli base +field-list --as user --base-token BASE_TOKEN --table-id TASK_TABLE_ID --format json
```

Workflow skeleton:

```json
{
  "client_token": "UNIQUE_CLIENT_TOKEN",
  "title": "任务总表变更自动写入更新记录",
  "status": "Enabled",
  "steps": [
    {
      "id": "task_changed",
      "type": "SetRecordTrigger",
      "title": "任务总表关键字段变更",
      "next": "add_update_log",
      "data": {
        "table_name": "任务总表",
        "record_watch_conjunction": "and",
        "record_watch_info": [],
        "field_watch_info": [
          { "field_name": "状态" },
          { "field_name": "最新进展" },
          { "field_name": "卡点/需老师确认" },
          { "field_name": "负责人" },
          { "field_name": "协作人" },
          { "field_name": "截止时间" },
          { "field_name": "交付物" },
          { "field_name": "研究计划" }
        ],
        "trigger_control_list": [],
        "condition_list": null
      }
    },
    {
      "id": "add_update_log",
      "type": "AddRecordAction",
      "title": "写入更新记录",
      "next": "sync_status_snapshot",
      "data": {
        "table_name": "更新记录",
        "field_values": [
          { "field_name": "提交标题", "value": [{ "value_type": "text", "value": "任务总表自动更新记录" }] },
          { "field_name": "关联任务", "value": [{ "value_type": "ref", "value": "$.task_changed.recordId" }] },
          { "field_name": "提交类型", "value": [{ "value_type": "option", "value": { "name": "进展更新" } }] },
          { "field_name": "提交内容", "value": [{ "value_type": "text", "value": "任务总表关键字段发生变更，已自动归档。" }] },
          { "field_name": "更新前状态", "value": [{ "value_type": "ref", "value": "$.task_changed.STATUS_SNAPSHOT_FIELD_ID" }] },
          { "field_name": "更新后状态", "value": [{ "value_type": "ref", "value": "$.task_changed.STATUS_FIELD_ID" }] },
          { "field_name": "提交人", "value": [{ "value_type": "ref", "value": "$.task_changed.recordModifiedUser" }] },
          { "field_name": "备注", "value": [{ "value_type": "text", "value": "由 Base Workflow 自动生成。" }] }
        ]
      }
    },
    {
      "id": "sync_status_snapshot",
      "type": "SetRecordAction",
      "title": "同步状态快照",
      "next": null,
      "data": {
        "table_name": "任务总表",
        "ref_info": { "step_id": "task_changed" },
        "field_values": [
          { "field_name": "状态快照", "value": [{ "value_type": "ref", "value": "$.task_changed.STATUS_FIELD_ID" }] }
        ]
      }
    }
  ]
}
```

Create and enable the workflow:

```powershell
lark-cli base +workflow-create --as user --base-token BASE_TOKEN `
  --json "@task-update-audit-workflow.json" --format json

lark-cli base +workflow-enable --as user --base-token BASE_TOKEN --workflow-id WORKFLOW_ID `
  --yes --format json
```

Important: test by editing one non-sensitive sample task. If bot/API/CLI writes do not trigger Workflow in this tenant, make the bot/API/CLI update path append a linked `更新记录` row itself.

## Add Records

Use batch create only after previewing rows with the user and receiving explicit upload/write confirmation. If the user revises the draft, render the revised table first and wait again.

```json
{
  "fields": [
    "Task Name",
    "Task Category",
    "Owner",
    "Collaborators",
    "Deadline",
    "Status",
    "Latest Progress",
    "Teacher Confirmation / Blocker",
    "Priority",
    "Deliverable",
    "Research Plan",
    "Related Materials"
  ],
  "rows": [
    [
      "Submit material group for assay",
      "Biochemical Submission",
      "Student B",
      "Student A",
      "2026-07-05 23:59:00",
      "Teacher Confirmation",
      null,
      "Confirm sample matrix and replicate number.",
      null,
      "Assay submission samples",
      "Use current material batch; confirm sampling stage before submission.",
      null
    ]
  ]
}
```

```powershell
lark-cli base +record-batch-create --as user --base-token BASE_TOKEN --table-id TABLE_ID `
  --json "@records.json" --format json
```

## Archive Updates

After a confirmed task update, update the main task row and add one linked row to `Update Log`.

Update the task row:

```powershell
lark-cli base +record-batch-update --as user --base-token BASE_TOKEN --table-id TASK_TABLE_ID `
  --json "@task-update.json" --format json
```

Create the update-log row:

```json
{
  "fields": [
    "提交标题",
    "关联任务",
    "提交类型",
    "提交内容",
    "更新前状态",
    "更新后状态",
    "提交人",
    "备注"
  ],
  "rows": [
    [
      "更新任务进展",
      [{ "id": "TASK_RECORD_ID" }],
      "进展更新",
      "本次更新的完整过程记录。",
      "未开始",
      "进行中",
      "Student A",
      null
    ]
  ]
}
```

```powershell
lark-cli base +record-batch-create --as user --base-token BASE_TOKEN --table-id UPDATE_LOG_TABLE_ID `
  --json "@update-log-record.json" --format json
```

## Send Group Message

Preview group messages with the user first when requested.

```powershell
lark-cli im +messages-send --as user --chat-id CHAT_ID --markdown "<message>" --format json
```

Mention a bot:

```text
<at id=BOT_OPEN_ID></at> message body
```
