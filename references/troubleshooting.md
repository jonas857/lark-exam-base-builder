# Troubleshooting

## Missing scope

Symptom:

```text
missing required scope(s): ...
```

Action:

- Ask the user to run the exact `lark-cli auth login --scope "..."` command suggested by the CLI.
- If the user runs the command locally and says authorization succeeded, continue.
- Do not retry repeatedly with the same failing command before authorization.

## Resource permission denied

Symptom:

```text
91403 you don't have permission
```

Action:

- Stop retrying.
- Explain this is resource-level permission, not merely OAuth scope.
- Ask the user to grant access/admin rights or choose a different location/account.

## Field name too long or formula references fail

Action:

- Switch to stable field names: `Q1`, `Q2`, ...
- Put full question text in field descriptions or the standard answer table.
- Use field IDs in formulas if supported/required.

## Select option mismatch

Symptoms:

- scoring returns 0 for a correct answer;
- formulas or filters cannot match option values.

Action:

- Trim accidental leading/trailing whitespace from all options.
- Preserve exactly the same display text in options and correct answers.
- Use `A、text` consistently, not sometimes `A. text` and sometimes `A、text`.
- For imported Excel data, normalize full-width/half-width punctuation before field creation.

## Multi-select formula too complex

Action:

1. Try exact set match formula from `scoring-formulas.md`.
2. If function support is unclear, create score fields as manual number fields and report that multi-select scoring requires manual configuration.
3. Alternatively, ask the user whether to simplify multi-select scoring or convert to single-choice questions.

## Formula field creation fails

Action:

- Read the current lark-base formula field guide if available.
- Escape quotes and newlines.
- Create formula fields one by one to identify the failing question.
- Fall back only for failing fields; do not abandon already-created resources.

## Old schema formula read issue

Symptom from `record-list`:

```text
formula field cannot be read through OpenAPI because this base uses an old schema version without backend formula computation
```

Action:

- Explain that the Feishu UI may still show formulas correctly.
- For a newly created Base, recommend checking whether backend formula calculation is enabled.
- Do not claim formulas are broken solely because OpenAPI cannot read them.

## Form question title is incomplete

Symptom:

```text
The form shows short field names such as Q1 分层抽样的主要目的 instead of the full question stem.
```

Cause:

- Stable answer field names are intentionally short for formula safety.
- Feishu forms may initially inherit the field name as the displayed question title.

Action:

- Run `+form-questions-list` to get question IDs.
- Run `+form-questions-update` and set each Q1~QN form question title to `Q{N}. {full_question}`.
- Set answer questions to required and use vertical option display when supported.

## Chinese JSON mojibake in form update

Symptom:

- Form question titles become garbled after `+form-questions-update` on Windows/Git Bash.

Action:

- Generate the `--questions` JSON with `json.dumps(..., ensure_ascii=True)`.
- Pass the escaped JSON string inline.
- Do not use `@file` for `--questions`; this flag expects an inline JSON string.

## Non-question helper field appears in the form

Symptom:

- A helper field such as `提交标识` appears in `答题试卷`.

Action:

- If it is non-required, the form remains usable.
- Ask the user before removing it from the form because deletion/hiding is a write operation on a persistent form resource.
- Use `+form-questions-delete` only after the user confirms the exact field/question ID.

## Form question order is not Q1~QN

Action:

- Check whether the current CLI/API exposes a sort/order field for form questions.
- If supported, update the form order to Q1~QN.
- If unsupported or unclear, disclose the limitation and ask the user to verify order manually in the Feishu UI.

## Dashboard group_by validation fails

Symptom:

- Dashboard ring/pie block creation fails validation for `group_by.sort`.

Action:

- Include both sort type and order:

```json
{
  "table_name": "答卷汇总表",
  "count_all": true,
  "group_by": [
    {
      "field_name": "是否及格",
      "sort": {
        "type": "group",
        "order": "asc"
      }
    }
  ]
}
```

## Form creation fails

Action:

- Keep tables and views.
- Report manual step:

```text
请在飞书页面打开《答卷汇总表》-> 创建表单《答题试卷》-> 仅展示 Q1~QN 题目字段，隐藏分数、结果、总分、是否及格、答案解析字段。
```

## Dashboard creation or data read fails

Action:

- Dashboard is optional; do not roll back the exam system.
- If block data returns `empty chart data`, report that metadata/config was created/read but computed chart data could not be fetched.
- The base tables/views remain usable.

## Batch limits and conflicts

- Batch record writes: <= 200 per batch.
- If `1254104`, split into smaller batches.
- If `1254291`, serialize writes and wait briefly before retrying.

## Avoid destructive actions

This skill creates new resources. It should not delete existing Base resources unless the user explicitly asks and confirms the exact target.
