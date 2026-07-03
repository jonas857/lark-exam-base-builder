# lark-cli command flow

Use `lark-cli base +...` shortcuts only. Always add `--as user` unless the user explicitly requests bot identity.

## Before executing

1. Parse and validate the question bank.
2. Prepare field JSON for both tables.
3. Prepare standard answer records.
4. For large JSON arguments, write files in the current working directory and pass them with relative `@./file.json` paths.
5. Show the creation plan and ask for confirmation.
6. Confirm authorization when CLI reports missing scopes.

## Useful help commands

When unsure about flags or JSON shape, run help first:

```bash
lark-cli base +base-create --help
lark-cli base +table-create --help
lark-cli base +field-create --help
lark-cli base +record-batch-create --help
lark-cli base +view-set-filter --help
lark-cli base +form-create --help
lark-cli base +dashboard-block-create --help
```

## Recommended creation order

### 1. Create Base with initial standard-answer table

Prefer creating the Base with `标准答案分值表` as the initial table:

```bash
lark-cli base +base-create \
  --name "<考试名称>管理微系统" \
  --table-name "标准答案分值表" \
  --fields @./standard_answer_fields.json \
  --as user
```

Capture:

- `base_token`
- initial standard answer `table_id`
- base URL if returned

If `--fields` is not supported or fails, create the Base first, then update/create fields explicitly.

### 2. Write standard answer records

Use batch create. Batch size must be <= 200.

```bash
lark-cli base +record-batch-create \
  --base-token <base_token> \
  --table-id <standard_answer_table_id> \
  --json @./exam_answers.json \
  --as user
```

Use `@./file.json` paths from the current working directory for large `--json` or `--fields` payloads. On Windows/Git Bash this is more reliable than absolute `/tmp/...` paths and avoids command-line truncation or quoting issues.

### 3. Create answer summary table

```bash
lark-cli base +table-create \
  --base-token <base_token> \
  --name "答卷汇总表" \
  --fields @./answer_fields.json \
  --as user
```

If creating all fields at once fails, create the table with a minimal field set, then add fields one by one or in safe groups:

1. system/basic fields;
2. answer select fields;
3. score formulas, using `--i-have-read-guide` when creating formula fields;
4. result formulas;
5. total and pass/fail formulas;
6. correct answer/explanation fields.

### 4. Create or configure views

Create views if command support is available; otherwise use existing default views and rename/update them.

Core views:

```text
答案与分值
答卷汇总
不及格名单
```

For `不及格名单`, set filter:

```json
{"logic":"and","conditions":[["是否及格","==","不及格"]]}
```

Command shape may vary; inspect help if needed:

```bash
lark-cli base +view-set-filter --help
```

Visible-field/order commands should be used after verifying names from `+field-list`.

### 5. Create and configure form

Create a form named `答题试卷` attached to `答卷汇总表`, then expose only answer fields.

After creation, list the form questions and update the displayed title of every Q1~QN answer question to the full question text. This is mandatory when answer table fields use short stable names: otherwise the form may show incomplete titles.

Known CLI details:

- `+form-questions-update --questions` expects an inline JSON string and does not support `@file`.
- On Windows/Git Bash, generate the JSON with `json.dumps(..., ensure_ascii=True)` before passing it to `--questions`; directly passing Chinese JSON may produce mojibake.
- Set answer questions to `required: true`.
- Prefer vertical option display, e.g. `option_display_mode: 1`, when supported.
- If helper fields such as `提交标识` appear in the form, ask the user before deleting them with `+form-questions-delete`.

If legacy/new form API mismatch occurs, do not fail the whole task. Report:

```text
表和字段已创建成功，但表单自动创建失败。请在飞书页面中基于《答卷汇总表》创建表单，并只展示 Q1~QN 题目字段。
```

### 6. Create dashboard

Create dashboard `考试看板` and blocks:

- `已提交答卷数`: count all records from `答卷汇总表`.
- `考试及格情况`: group by `是否及格`, count all. For ring/pie-style blocks, include `sort.type` and `sort.order` in `group_by`, for example `{ "field_name": "是否及格", "sort": { "type": "group", "order": "asc" } }`.

If dashboard creation fails, keep the rest of the system and report the failure.

## Authorization handling

If CLI returns `missing_scope`, guide the user to run the exact suggested auth command. Example:

```bash
! lark-cli auth login --scope "base:dashboard:read base:dashboard:write"
```

If CLI returns `91403` or resource-level permission denied, do not loop. Tell the user they need access/admin permission to the target resource.

## Verification after creation

At minimum, verify:

```bash
lark-cli base +table-list --base-token <base_token> --as user
lark-cli base +field-list --base-token <base_token> --table-id <answer_table_id> --as user
lark-cli base +record-list --base-token <base_token> --table-id <standard_answer_table_id> --limit 3 --as user
```

If possible, create one test submission record or ask the user whether to create a test record. Do not write test data without permission.

## Reporting partial completion

Use a clear status table:

| Resource | Status | Notes |
|---|---|---|
| Base | created | URL/token |
| 标准答案分值表 | created | N records |
| 答卷汇总表 | created | M fields |
| 表单 | failed/skipped/created | reason |
| 看板 | failed/skipped/created | reason |
