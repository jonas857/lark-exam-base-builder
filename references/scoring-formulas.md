# Scoring formulas

Use hybrid formula mode by default: the standard answer table stores answers for humans, and the answer summary table computes scores directly with formulas.

## Formula generation principles

1. Generate one score formula per question: `分数 N`.
2. Generate one result formula per question: `{N}⃣️答题结果`.
3. Generate total score formula: `总分`.
4. Generate pass/fail formula: `是否及格`.
5. Escape quotes and special characters in option text.
6. Prefer field IDs in final formulas if lark-cli requires them or if names contain difficult characters.
7. Test formula creation with one question first when possible, then batch the rest.
8. When creating formula fields through `lark-cli base +field-create`, include `--i-have-read-guide` even if the flag is not shown prominently in help output.

## Single choice / judgment scoring

Conceptual formula:

```text
IF({Q1 飞书是什么？} = "D、一站式企业协作平台", 10, 0)
```

Judgment questions are equivalent:

```text
IF({Q9 助手置顶是否有帮助？} = "正确", 10, 0)
```

## Multiple choice scoring

Default rule: exact set match, otherwise 0.

The generated formula should ensure:

- all correct options are selected;
- no incorrect options are selected;
- the selection count equals the correct answer count, if a reliable count function exists.

Conceptual formula:

```text
IF(
  AND(
    CONTAINS({Q7 快捷表情有哪些好处？}, "A、简单快捷"),
    CONTAINS({Q7 快捷表情有哪些好处？}, "B、不会触发群内其他人的通知"),
    CONTAINS({Q7 快捷表情有哪些好处？}, "C、不会刷屏"),
    NOT(CONTAINS({Q7 快捷表情有哪些好处？}, "D、可以让沟通更加严肃正式"))
  ),
  10,
  0
)
```

Because Feishu Base formula syntax can vary across schema versions, verify the exact function names using the current `lark-cli base +field-create` guide before creating formula fields. If a robust exact-set formula is not possible, use a simpler safe fallback and disclose it:

- formula score fields may be created as manual number fields; or
- multiple-choice questions may be marked for manual review; or
- use only single-choice/judgment questions in v1 creation.

## Verified lark-cli formula style

In tested Bases, formulas created through `lark-cli` may need explicit table and field IDs instead of field names. A working equality formula shape is:

```text
IF(bitable::$$table[tblUxbohsmmuSiWu].$$field[fldCeYDlm9]="B、在各关键子群体中具有代表性",4.545,0)
```

A working total formula shape is:

```text
SUM(bitable::$$table[tblUxbohsmmuSiWu].$$field[fldScore1], bitable::$$table[tblUxbohsmmuSiWu].$$field[fldScore2])
```

Use actual created field IDs from `+field-list`.

## Result formula

For question N with full score S:

```text
IF({分数 N} = S, "✅", "❌")
```

## Total score formula

```text
SUM({分数 1}, {分数 2}, ..., {分数 N})
```

For many questions, build this mechanically from the actual created score field names or IDs.

## Score display strategy

If the source question bank already contains scores, preserve them. If scores must be generated automatically, avoid surprising totals such as `4.545 × 22 = 99.99` when the user expects 100.

Recommended options:

1. Use 1 point per question and make the total score equal to the question count.
2. Use a rounded total formula such as `ROUND(SUM(...), 2)` when supported.
3. If the user requires exactly 100 points, distribute scores mechanically and put the rounding remainder on the final question.

Always show the computed total score in the creation plan before creating resources.

## Pass/fail formula

```text
IF({总分} >= passing_score, "及格", "不及格")
```

## Correct answer and explanation fields

If using formula fields for fixed text:

```text
"D、一站式企业协作平台"
```

```text
"飞书是一站式企业协作平台。"
```

If formula creation for fixed text is inconvenient, create text fields and optionally prefill only via records later; for an answer summary table, formula fixed text is preferred because every submission row receives the same answer/explanation automatically.

## Escaping rules

Before embedding text in formulas:

- Replace `"` with escaped quotes according to Feishu formula requirements.
- Normalize newlines in answer/explanation text or replace with spaces if formulas do not allow raw newlines.
- Trim accidental leading/trailing option whitespace.
- Preserve meaningful punctuation such as `、`, `\`, parentheses, and Chinese punctuation.

## Old schema warning

Some old Bases cannot return formula results through OpenAPI. Newly created Bases should generally be better, but still report if `record-list` says formula fields are unsupported. This does not necessarily mean the formula is broken in the Feishu UI.
