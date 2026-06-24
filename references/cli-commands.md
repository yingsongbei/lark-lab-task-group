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

## Create Views

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

## Add Records

Use batch create after previewing rows with the user:

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

## Send Group Message

Preview group messages with the user first when requested.

```powershell
lark-cli im +messages-send --as user --chat-id CHAT_ID --markdown "<message>" --format json
```

Mention a bot:

```text
<at id=BOT_OPEN_ID></at> message body
```
