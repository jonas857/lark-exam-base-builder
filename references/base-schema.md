# Generated Base schema

This reference defines the default Base structure created by `lark-exam-base-builder`.

## Resource tree

```text
{考试名称}管理微系统
├── 答卷汇总表
│   ├── 答卷汇总
│   ├── 不及格名单
│   └── 答题试卷
├── 标准答案分值表
│   └── 答案与分值
└── 考试看板
    ├── 已提交答卷数
    └── 考试及格情况
```

## Table: 标准答案分值表

Purpose: store the normalized question bank, answers, scores, and explanations.

Default fields:

| Field | Type | Required | Notes |
|---|---|---|---|
| 题号 | number | yes | question order |
| 题型 | select | yes | 单选 / 多选 / 判断 |
| 题目 | text | yes | full question title |
| 选项 | text | yes for choice questions | one option per line; display form such as `A、...` |
| 正确答案 | text | yes | joined answer display values, e.g. `A、...；B、...` |
| 分值 | number | yes | positive number |
| 解析 | text | no | answer explanation |

Use text for `正确答案` in v1 for stability. The standard answer table is the source of truth for humans, while scoring formulas are generated in the answer table.

## Table: 答卷汇总表

Purpose: receive form submissions, compute scores, show results.

Base fields:

| Field | Type | Notes |
|---|---|---|
| 考试时间 | datetime or created_time | prefer created/submitted time if available; otherwise datetime |
| 考试人（自动获取） | created_by | automatic submitter |
| 总分 | formula | SUM of all `分数 N` fields |
| 是否及格 | formula | `IF(总分 >= passing_score, "及格", "不及格")` |

Per question fields for question `N`:

| Field pattern | Type | Notes |
|---|---|---|
| `Q{N} {short_title}` or original title | select | answer input field; `multiple=true` for multiple choice |
| `分数 {N}` | formula | computed score |
| `{N}⃣️答题结果` | formula | `✅` if full score else `❌` |
| `问题{N}正确答案` | formula/text | fixed correct answer text |
| `问题{N}答案解析` | formula/text | fixed explanation text |

## Field naming modes

### Stable mode, default

Use concise, formula-safe answer field names:

```text
Q1 飞书是什么？
Q2 关于全局搜索...
Q3 紧急事项沟通...
```

Rules:

- Prefix every answer field with `Q{index}`.
- If full title is too long, use `Q{index}` as field name and put full question in field description.
- Trim accidental whitespace from options.
- Preserve display prefixes (`A、`, `B、`) consistently across option values and answers.

### Original-title mode

Use the full question as the field name. This is closer to the source Base but more fragile. Use only if the user explicitly prefers it.

## Views

### 答案与分值

Table: `标准答案分值表`

Visible fields:

```text
题号
题型
题目
选项
正确答案
分值
解析
```

### 答卷汇总

Table: `答卷汇总表`

No filter. Visible fields should follow:

```text
考试时间
考试人（自动获取）
总分
是否及格
Q1 ...
分数 1
1⃣️答题结果
问题1正确答案
问题1答案解析
Q2 ...
分数 2
2⃣️答题结果
问题2正确答案
问题2答案解析
...
```

### 不及格名单

Table: `答卷汇总表`

Filter:

```json
{
  "logic": "and",
  "conditions": [["是否及格", "==", "不及格"]]
}
```

Visible fields should prioritize:

```text
考试时间
考试人（自动获取）
总分
是否及格
all question answer fields
all score/result fields
correct answer/explanation fields
```

## Form: 答题试卷

Description template:

```text
本考试共 {question_count} 题，满分 {total_score} 分，{passing_score} 分及格。
请根据题目要求完成作答。提交后系统将自动计算成绩。
```

### Form question titles

When stable field naming is used, answer field names may be shortened, for example `Q1 分层抽样的主要目的`. Do not let the form display only these shortened field names.

After creating the form, call `+form-questions-update` and update every answer question title to the full question text:

```text
Q{N}. {full_question}
```

For each answer question, set:

- `required: true`
- vertical option display when supported, e.g. `option_display_mode: 1`

Show only answer fields (`Q1...QN`). Hide scoring/result/system fields. If a non-question helper field such as `提交标识` appears in the form, ask the user before deleting it from the form.

If form automation fails, report the table is ready and instruct the user to create a form from `答卷汇总表`, exposing only answer fields.

## Dashboard: 考试看板

Default blocks:

### 已提交答卷数

```json
{
  "table_name": "答卷汇总表",
  "count_all": true
}
```

### 考试及格情况

```json
{
  "table_name": "答卷汇总表",
  "count_all": true,
  "group_by": [
    {
      "field_name": "是否及格",
      "sort": { "type": "group", "order": "asc" }
    }
  ]
}
```

Dashboard creation is useful but non-critical. If it fails, do not roll back tables.
