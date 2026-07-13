# Robust Weekly Report Automation

Use this reference when a group bot or connected agent runs a scheduled lab-task report.

## Required Inputs

Resolve these values from real API responses and deployment settings:

- `BASE_TOKEN`
- `TASK_TABLE_ID`
- `THIS_WEEK_VIEW_ID`
- `TEACHER_CONFIRMATION_VIEW_ID`
- `RECENT_COMPLETED_VIEW_ID`
- `TARGET_CHAT_ID`
- `TIMEZONE`
- User-selected weekday and time

Never commit real tokens, IDs, names, usernames, or project identifiers to the reusable skill.

## Three-View Data Contract

The report reads exactly these views:

1. `This Week` / `本周任务`
2. `Teacher Confirmation` / `待老师确认`
3. `Recent 4 Weeks Completed` / `近四周已完成`

Use dynamic formulas or relative-date conditions. Do not freeze any view to the week in which the tracker was created.

Read every page until `has_more=false`. Use the CLI's full option names, including `--format json`; do not use ambiguous shorthand such as `-o`.

## This-Week Qualification Contract

Treat `This Week` as a dated commitment list, not a synonym for all open work:

- Use the configured timezone.
- Include only records with an explicit deadline from the current Monday 00:00 inclusive to the next Monday 00:00 exclusive.
- Exclude `Completed` records; completed work belongs in the rolling completed section.
- Exclude records with blank deadlines or deadlines outside the current week.
- Never infer membership from free text in `Latest Progress`, `Research Plan`, notes, or phrases such as "planned this week".
- Revalidate records after reading the view so a stale or misconfigured view cannot inflate the report.

For long-running experiments, `Deadline` should normally mean the current stage's committed completion date. Teams may split a project into dated stage tasks when one row cannot represent the next deliverable clearly. Members should update tasks continuously and, at minimum, review deadlines before the scheduled weekly report. Ask the user to choose the local cutoff; for a Monday evening report, a cutoff shortly before generation is usually practical.

## Prompt Construction Rules

- Start with a direct action instruction such as `请执行以下每周任务`.
- Do not write a long role description that resembles a system prompt.
- Replace the scheduled task's entire prompt in one update. Do not split the prompt across multiple chat messages.
- Include exact Base/table/view/chat identifiers only in the deployed prompt, not in this repository.
- State that updating the schedule must not immediately run the report.
- Require all three reads to succeed before sending anything to the group.
- Send one concise group message, not one message per view.
- Add a weekly deduplication marker such as `lab-weekly-YYYY-Www`; skip sending when that marker has already been used for the same target chat and week.
- If displaying a week number, calculate the ISO-8601 week from the report date in the configured timezone. Do not copy the previous reporting period's week number.

## Reusable Chinese Prompt

```text
请执行以下每周任务：读取指定飞书多维表格的三个视图，整理实验室任务周报，并在全部数据读取成功后向目标群发送一条消息。

【执行时间】
每周 <WEEKDAY> <TIME>，时区 <TIMEZONE>。修改本定时任务时不要立即执行。

【数据源】
Base：<BASE_TOKEN>
任务表：<TASK_TABLE_ID>
本周任务视图：<THIS_WEEK_VIEW_ID>
待老师确认视图：<TEACHER_CONFIRMATION_VIEW_ID>
近四周已完成视图：<RECENT_COMPLETED_VIEW_ID>
目标群：<TARGET_CHAT_ID>

【读取规则】
依次读取以上三个视图，使用 lark-cli 的完整参数名和 --format json，不使用 -o 等缩写。处理全部分页，直到 has_more=false。任务字段至少读取：任务名称、任务类别、负责人、协作人、截止时间、状态、最新进展、卡点/需老师确认、优先级、交付物、研究计划和更新时间。

【本周任务判定】
“本周任务”仅保留截止时间明确落在当前自然周周一 00:00（含）至下周一 00:00（不含）、且状态不是“已完成”的任务。截止时间为空或不在本周的任务不得进入本周任务。不得根据最新进展、研究计划或“本周完成”等自由文本推断日期。读取视图后仍需逐条复核日期和状态，防止视图配置失效。若显示周数，按当前时区的 ISO-8601 自然周计算。

【周报结构】
1. 本周任务：列出任务名称、负责人、截止时间、状态和最新进展。
2. 待老师确认：列出任务名称、负责人及需要确认的具体事项。
3. 近四周已完成：列出任务名称、负责人和最终进展。
4. 本周提醒：仅根据以上三个视图生成简洁提醒，不从任务总表补充其他任务。

【发送条件】
只有三个视图全部读取成功且数据结构有效时，才向目标群发送一条简洁周报。发送前检查本周去重标记 lab-weekly-YYYY-Www；同一目标群同一周已发送则停止，不重复发送。

【失败处理】
任一读取、解析或发送准备步骤失败时，不向群里发送残缺周报。在定时任务运行结果中记录失败步骤、错误摘要和建议重试方式，不要把内部 token、ID、命令参数或错误堆栈发到群里。

【完成后】
返回本次读取的三个视图、任务数量、发送结果和去重标记。
```

## Weekly Message Shape

```text
【实验室任务周报】<YYYY-MM-DD>

一、本周任务
1. ...

二、待老师确认
1. ...

三、近四周已完成
1. ...

四、本周提醒
1. ...
```

Omit empty numbered items or write `无`, according to the user's preference. Keep the message compact and avoid exposing internal identifiers.

## Validation

After replacing the scheduled prompt:

1. Read the scheduled-task definition back and confirm the full prompt is present.
2. Confirm the configured schedule and timezone.
3. Confirm no shorthand CLI flags remain.
4. Run one manual test only when the user approves sending or when the test uses a non-production chat.
5. Verify that a second run in the same week does not send a duplicate.
6. Verify that blank-deadline, out-of-week, and completed records are absent from `This Week`, even if a source view accidentally returns them.
7. Verify the displayed ISO week number against the report date and configured timezone.
