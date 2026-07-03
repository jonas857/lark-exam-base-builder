# lark-exam-base-builder

A Claude Code skill for building a Feishu/Lark Base exam management micro-system from a question bank.

This skill turns Markdown, CSV, Excel-style, or pasted exam questions into a Lark Base structure with:

- a standard answer and score table;
- an answer submission table;
- an answer form;
- automatic per-question scoring formulas;
- total score and pass/fail calculation;
- answer summary and failing-student views;
- an optional exam dashboard.

## What it creates

```text
{Exam Name}管理微系统
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

## Supported question types

| Type | Canonical value | Form field | Scoring |
|---|---|---|---|
| 单选 | `single` | select, single choice | exact match |
| 多选 | `multiple` | select, multiple choice | exact set match where supported |
| 判断 | `boolean` | select, single choice | exact match: 正确/错误 |

Unsupported question types, such as fill-in-the-blank or essay questions, should be skipped, converted to manual scoring, or handled by a future normalized schema.

## Requirements

- Claude Code
- `lark-cli`
- A Feishu/Lark account authenticated in `lark-cli`
- Permissions to create and edit Lark Base resources

The skill is designed around `lark-cli base +...` shortcut commands, for example:

```bash
lark-cli base +base-create
lark-cli base +table-create
lark-cli base +record-batch-create
lark-cli base +field-create
lark-cli base +form-create
lark-cli base +form-questions-update
lark-cli base +dashboard-create
```

## Installation

Clone this repository into your Claude Code skills directory:

```bash
git clone https://github.com/jonas857/lark-exam-base-builder.git ~/.claude/skills/lark-exam-base-builder
```

Restart Claude Code or reload skills if needed.

## Usage examples

Ask Claude Code something like:

```text
根据这份题库创建飞书考试系统
```

```text
把这个 Markdown 题库变成飞书多维表格考试表单，并自动判分
```

```text
创建企业培训考试管理微系统，包含答卷汇总、不及格名单和考试看板
```

The skill should then:

1. parse the question bank into the canonical schema;
2. validate question numbers, options, answers, scores, and pass line;
3. show a creation plan;
4. ask for confirmation before creating persistent Feishu/Lark resources;
5. create Base tables, fields, views, form, formulas, and dashboard;
6. report created URLs/IDs and any partial failures.

## Input format

See:

- [`references/exam-question-format.md`](references/exam-question-format.md)
- [`references/examples/sample-questions.md`](references/examples/sample-questions.md)
- [`references/examples/sample-questions.csv`](references/examples/sample-questions.csv)

Minimal Markdown example:

```markdown
# 飞书培训考试题库

考试名称：飞书培训考试
及格线：60

## 1. 飞书是什么？
题型：单选
分值：10

A. 聊天软件
B. 视频会议工具
C. 在线文档平台
D. 一站式企业协作平台

答案：D
解析：飞书是一站式企业协作平台。
```

## Design notes

The default implementation uses **hybrid formula mode**:

- `标准答案分值表` stores the normalized question bank, answers, scores, and explanations for humans.
- `答卷汇总表` computes scores directly with generated formula fields.

This is more robust for automated generation than lookup-based scoring, especially when dealing with select options and multi-select answers.

## Important implementation details

The reference docs include several tested operational details:

- use `lark-cli base +...` shortcuts only;
- add `--as user` to Base commands unless bot identity is explicitly requested;
- use relative `@./file.json` paths for large `--json` and `--fields` payloads;
- `+form-questions-update --questions` expects inline JSON and does not support `@file`;
- on Windows/Git Bash, generate inline form JSON with `ensure_ascii=True` to avoid Chinese mojibake;
- formula field creation may require `--i-have-read-guide`;
- answer table field names may be short, but form titles must be updated to full question stems;
- dashboard `group_by.sort` should include both `type` and `order`.

## Documentation map

- [`SKILL.md`](SKILL.md) — Claude Code skill entrypoint and workflow.
- [`references/base-schema.md`](references/base-schema.md) — generated Base schema.
- [`references/exam-question-format.md`](references/exam-question-format.md) — accepted input formats and canonical JSON.
- [`references/scoring-formulas.md`](references/scoring-formulas.md) — formula strategy and tested formula shapes.
- [`references/lark-cli-command-flow.md`](references/lark-cli-command-flow.md) — command order and CLI caveats.
- [`references/troubleshooting.md`](references/troubleshooting.md) — known failures and recovery steps.

## Safety

This skill creates persistent Feishu/Lark resources. It should always ask for explicit confirmation before creating resources and should not delete existing Base resources unless the user confirms the exact target.

## License

MIT. See [`LICENSE`](LICENSE).
