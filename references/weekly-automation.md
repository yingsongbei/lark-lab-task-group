# Robust Weekly Report Automation

Use this reference when a group bot or connected agent runs a scheduled lab-task report.

## Required Inputs

Resolve these values from real API responses and deployment settings:

- `BASE_TOKEN`
- `TASK_TABLE_ID`
- `THIS_WEEK_VIEW_ID`
- `OVERDUE_VIEW_ID`
- `TEACHER_CONFIRMATION_VIEW_ID`
- `RECENT_COMPLETED_VIEW_ID`
- `TARGET_CHAT_ID`
- `TIMEZONE`
- User-selected weekday and time

Never commit real tokens, IDs, names, usernames, or project identifiers to the reusable skill.

## Four-View Data Contract

The report reads exactly these views:

1. `This Week` / `本周任务`
2. `Overdue Unfinished` / `逾期未完成`
3. `Teacher Confirmation` / `待老师确认`
4. `Recent 4 Weeks Completed` / `近四周已完成`

Use dynamic formulas or relative-date conditions. Do not freeze any view to the week in which the tracker was created.

Read every page until `has_more=false`. Use the CLI's full option names, including `--format json`; do not use ambiguous shorthand such as `-o`.

## Prompt Construction Rules

- Start with a direct action instruction such as `请执行以下每周任务`.
- Do not write a long role description that resembles a system prompt.
- Replace the scheduled task's entire prompt in one update. Do not split the prompt across multiple chat messages.
- Include exact Base/table/view/chat identifiers only in the deployed prompt, not in this repository.
- State that updating the schedule must not immediately run the report.
- Require all four reads to succeed before sending anything to the group.
- Send one concise group message, not one message per view.
- Add a weekly deduplication marker such as `lab-weekly-YYYY-Www`; skip sending when that marker has already been used for the same target chat and week.

## Reusable Chinese Prompt

```text
请执行以下每周任务：读取指定飞书多维表格的四个视图，整理实验室任务周报，并在全部数据读取成功后向目标群发送一条消息。

【执行时间】
每周 <WEEKDAY> <TIME>，时区 <TIMEZONE>。修改本定时任务时不要立即执行。

【数据源】
Base：<BASE_TOKEN>
任务表：<TASK_TABLE_ID>
本周任务视图：<THIS_WEEK_VIEW_ID>
逾期未完成视图：<OVERDUE_VIEW_ID>
待老师确认视图：<TEACHER_CONFIRMATION_VIEW_ID>
近四周已完成视图：<RECENT_COMPLETED_VIEW_ID>
目标群：<TARGET_CHAT_ID>

【读取规则】
依次读取以上四个视图，使用 lark-cli 的完整参数名和 --format json，不使用 -o 等缩写。处理全部分页，直到 has_more=false。任务字段至少读取：任务名称、任务类别、负责人、协作人、截止时间、状态、最新进展、卡点/需老师确认、优先级、交付物、研究计划和更新时间。

【周报结构】
1. 本周任务：列出任务名称、负责人、截止时间、状态和最新进展。
2. 逾期未完成：列出任务名称、负责人、原截止时间和当前进展。
3. 待老师确认：列出任务名称、负责人及需要确认的具体事项。
4. 近四周已完成：列出任务名称、负责人和最终进展。
5. 待补充进度：指出长期没有更新或信息不完整的任务。

【发送条件】
只有四个视图全部读取成功且数据结构有效时，才向目标群发送一条简洁周报。发送前检查本周去重标记 lab-weekly-YYYY-Www；同一目标群同一周已发送则停止，不重复发送。

【失败处理】
任一读取、解析或发送准备步骤失败时，不向群里发送残缺周报。在定时任务运行结果中记录失败步骤、错误摘要和建议重试方式，不要把内部 token、ID、命令参数或错误堆栈发到群里。

【完成后】
返回本次读取的四个视图、任务数量、发送结果和去重标记。
```

## Weekly Message Shape

```text
【实验室任务周报】<YYYY-MM-DD>

一、本周任务
1. ...

二、逾期未完成
1. ...

三、待老师确认
1. ...

四、近四周已完成
1. ...

五、待补充进度
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
