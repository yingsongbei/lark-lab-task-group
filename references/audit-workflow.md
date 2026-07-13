# Source-Aware Update Log Workflow

Use this pattern when members edit `Task Register` directly and CLI/bot writes must remain distinguishable from human edits.

## Deployment Inputs

Resolve these values at deployment time. Never commit them to a reusable skill:

- `BASE_TOKEN`
- `TASK_TABLE_ID`
- `UPDATE_LOG_TABLE_ID`
- `COORDINATOR_OPEN_ID`
- `COORDINATOR_DISPLAY_NAME`
- `MACHINE_SOURCE_LABEL`, such as `Feishu CLI` or `Lark CLI`
- Real field IDs returned by `field-list`

Ask the user which people and service accounts may repair the audit table. Ordinary members should be read-only in `Update Log`.

## Internal Fields

Add two text fields to `Task Register` and hide both from every routine view:

1. `Status Snapshot` / `状态快照`: previous status source for the next audit row.
2. `Update Source` / `更新来源`: temporary machine-write marker.

CLI/bot task writes must set `Update Source = MACHINE_SOURCE_LABEL` in the same task patch. Human edits must not write this field.

Do not include either internal field in the trigger's watched-field list. Otherwise the cleanup action can retrigger the workflow.

## Stable Topology

Use this topology:

```text
Task changed
  -> Is Update Source the machine label?
       true  -> Add machine update log -> Sync machine snapshot and clear source
       false -> Is modifier the configured coordinator?
                  true  -> Add coordinator update log -> Sync coordinator snapshot and clear source
                  false -> Add other-member update log -> Sync other-member snapshot and clear source
```

Use three independent `SetRecordAction` cleanup steps. In nested Feishu/Lark Workflow branches, the service may silently discard multiple branch links to one shared cleanup node even when the update response echoes those links.

Recommended stable step IDs:

- `trigger_task_change`
- `source_branch`
- `add_machine_log`
- `sync_machine_snapshot`
- `manual_user_branch`
- `add_coordinator_log`
- `sync_coordinator_snapshot`
- `add_other_manual_log`
- `sync_other_snapshot`

Each log action points only to its matching cleanup action. Each cleanup action writes the same two values to the triggering task:

- `Status Snapshot = current Status`
- `Update Source = empty text`

## Attribution Rules

Write user-facing audit fields as follows:

| Source | Submitted By | Notes |
|---|---|---|
| CLI/bot write | `MACHINE_SOURCE_LABEL` | Generated after a confirmed CLI/bot task update. |
| Coordinator manual edit | `COORDINATOR_DISPLAY_NAME` | Generated after the coordinator manually changed the task table. |
| Other member manual edit | Actual `recordModifiedUser` display value | Generated after that member manually changed the task table. |

Do not display numeric account aliases, open IDs, user IDs, Base tokens, or internal source-marker values in audit rows. Stable IDs belong only in Workflow conditions.

If repairing older history, change only incorrect attribution/source notes unless the user explicitly asks to change the historical content. Preserve meaningful notes such as whether a confirmation point was cleared or retained.

## Trigger And Log Contents

Watch only user-facing task fields that matter to the team, normally:

- Task name and category
- Owner and collaborators
- Deadline
- Status
- Latest progress
- Teacher confirmation / blocker
- Priority
- Deliverable
- Research plan
- Related materials

Each generated log row should include:

- Linked task record
- Update type
- Current task summary
- Previous status from `Status Snapshot`
- New status from current `Status`
- Source-aware submitter
- Source-aware note
- Created time

The generic Workflow may label the update type `Progress Update` when the platform cannot identify exactly which watched field changed. Do not claim a status change merely because the record was edited.

## Save And Readback Validation

Do not trust only the `workflow-update` response. After every update:

1. Call `workflow-get` independently.
2. Confirm the workflow status is enabled.
3. Confirm `add_machine_log.next = sync_machine_snapshot`.
4. Confirm `add_coordinator_log.next = sync_coordinator_snapshot`.
5. Confirm `add_other_manual_log.next = sync_other_snapshot`.
6. Confirm every cleanup action writes both `Status Snapshot` and an empty `Update Source`.
7. Confirm the two internal fields remain hidden from all routine views.

Test each source path with a non-sensitive sample only when the user accepts the resulting audit rows. Structural readback is preferable when a live test would create noise.

## API/CLI Fallback

Some tenants do not trigger Base Workflow for API or bot writes. Test this once during setup.

If machine writes do not trigger the Workflow, the confirmed machine-write operation must perform all four actions together:

1. Update the task's current fields.
2. Append the linked `Update Log` row with the machine label.
3. Sync `Status Snapshot` to current status.
4. Clear `Update Source`.

Never leave the machine source marker populated. A stale marker can cause the next human edit to be attributed to the CLI/bot.
