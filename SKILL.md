---
name: lark-exam-base-builder
license: MIT
metadata:
  version: "0.1.0"
  category: lark-base
  author: local
  depends_on:
    - lark-cli
    - lark-base
    - lark-sheets
    - minimax-docx
    - minimax-xlsx
description: >
  Build a Feishu/Lark Base exam management micro-system from a question bank. Use this skill when the user wants to upload or paste questions and have Claude create a Base with question bank, answer submission table, auto scoring, pass/fail views, form, and dashboard using lark-cli base shortcuts.
triggers:
  - 飞书考试
  - 多维表格考试
  - 考试管理微系统
  - 题库生成表单
  - 自动判分
  - 答卷汇总
  - 不及格名单
  - Lark exam Base
  - Feishu exam Base
---

# lark-exam-base-builder

Create a Feishu/Lark Base exam management micro-system from a question bank.

## When to use

Use this skill when the user wants to:

- create a Feishu/Lark Base exam system from a question bank;
- turn Excel/CSV/Markdown/text questions into a Base and answer form;
- build an enterprise training exam management micro-system;
- generate answer summary, automatic scoring, pass/fail list, or exam dashboard;
- replicate the design logic of the existing Base named `企业培训考试管理微系统`.

Do **not** use this skill when the user only wants generic question analysis, ordinary questionnaire creation without scoring, or small edits to an unrelated existing Base.

## Core design model

The generated system follows this model:

```text
Question bank input
  -> 标准答案分值表
  -> 答题试卷 form
  -> 答卷汇总表
  -> per-question score/result formulas
  -> 总分 / 是否及格
  -> 答卷汇总 view
  -> 不及格名单 view
  -> 考试看板
```

Default resources:

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

## Mandatory workflow

1. **Parse the question bank** into the canonical schema in `references/exam-question-format.md`.
2. **Validate before creating anything**. Check missing answers, invalid options, duplicate question numbers, score totals, and question count.
3. **Explain the creation plan and ask for confirmation** because creating Base resources is outward-facing and persistent.
4. **Use `lark-cli base +...` shortcuts only** for Base operations. Do not use legacy aggregate commands.
5. **Default to user identity**: add `--as user` to all Base commands unless the user explicitly requests bot identity.
6. **Create resources in dependency order** from `references/lark-cli-command-flow.md`.
7. **After form creation, update form questions** so each question title uses the full question text, not just the shortened answer field name.
8. **Report partial failures honestly**. If dashboard/form creation fails but tables succeeded, say exactly what was created and what remains manual.

## Default implementation mode

Use **hybrid formula mode** by default:

- `标准答案分值表` stores the normalized question bank, answers, scores, and explanations.
- `答卷汇总表` uses generated formula fields to score each question directly.

Do not default to Lookup-based scoring. Lookup scoring matches the source Base conceptually, but is more fragile for automatic generation, especially for multi-select questions.

## Supported v1 question types

| Type | Canonical value | Answer field | Scoring |
|---|---|---|---|
| 单选 | `single` | select, `multiple=false` | exact match |
| 多选 | `multiple` | select, `multiple=true` | exact set match; otherwise 0 |
| 判断 | `boolean` | select, `multiple=false` | exact match: 正确/错误 |

For unsupported types such as fill-in-blank or essay, stop and ask whether to skip them, convert them to manual scoring, or postpone.

## Default configuration

If the user does not specify values:

```json
{
  "exam_name": "企业培训考试",
  "passing_score": "60% of total score, rounded sensibly",
  "scoring_mode": "exact_match",
  "allow_resubmit": true,
  "include_explanations": true,
  "create_form": true,
  "create_dashboard": true,
  "field_naming": "stable",
  "max_questions_for_wide_table": 50
}
```

If there are more than 50 questions, warn that the one-question-one-field table will be very wide. For 80+ questions, recommend a normalized `题库表 + 答卷表 + 答题明细表` design instead of v1 wide-table generation.

## References to read when performing the task

Read these files as needed:

- `references/exam-question-format.md` — accepted input formats and canonical JSON.
- `references/base-schema.md` — generated Base, table, field, view, form, and dashboard schema.
- `references/scoring-formulas.md` — scoring formula strategy and escaping rules.
- `references/lark-cli-command-flow.md` — safe command order and recovery.
- `references/troubleshooting.md` — common failures and fallbacks.
- `references/examples/sample-questions.md` — minimal Markdown question bank.
- `references/examples/sample-questions.csv` — CSV question bank format.

## Creation confirmation template

Before creating resources, show a plan like:

```text
我将创建以下飞书 Base 考试系统：

Base：{exam_name}管理微系统
题目数量：{n}
题型分布：单选 {single} / 多选 {multiple} / 判断 {boolean}
总分：{total_score}
及格线：{passing_score}
判分规则：完全匹配得分，否则 0 分

将创建：
- 表：标准答案分值表、答卷汇总表
- 记录：{n} 条标准答案记录
- 视图：答案与分值、答卷汇总、不及格名单
- 表单：答题试卷
- 看板：考试看板（已提交答卷数、考试及格情况）

是否继续？
```

Only proceed after explicit user confirmation.

## Final output template

After creation, report:

```text
已创建完成：
- Base：{base_url}
- 标准答案分值表：{table_id/name}
- 答卷汇总表：{table_id/name}
- 答题试卷：{form_id/link if available}
  - 已将表单题目标题更新为完整题干；请最终检查题目顺序和是否仍有非题目辅助字段。
- 答卷汇总视图：{view_id}
- 不及格名单视图：{view_id}
- 考试看板：{dashboard_id/link if available}

维护说明：
- 修改题目/答案后，需要同步更新答卷表的判分公式。
- 多选题默认完全一致才得分。
- 如公式或表单自动创建失败，见下方待手动完成事项。
```
