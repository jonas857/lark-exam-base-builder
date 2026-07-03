# Exam question format

This skill accepts Markdown, plain text, CSV, and Excel-style question banks. Always normalize inputs into the canonical JSON schema before creating any Base resources.

## Canonical JSON schema

```json
{
  "exam": {
    "name": "飞书培训考试",
    "passing_score": 60,
    "allow_resubmit": true,
    "scoring_mode": "exact_match",
    "show_answer_after_submit": false
  },
  "questions": [
    {
      "index": 1,
      "type": "single",
      "question": "飞书是什么？",
      "options": [
        { "key": "A", "text": "聊天软件", "display": "A、聊天软件" },
        { "key": "B", "text": "视频会议工具", "display": "B、视频会议工具" },
        { "key": "C", "text": "在线文档平台", "display": "C、在线文档平台" },
        { "key": "D", "text": "一站式企业协作平台", "display": "D、一站式企业协作平台" }
      ],
      "answer_keys": ["D"],
      "answer_values": ["D、一站式企业协作平台"],
      "score": 10,
      "explanation": "飞书是一站式企业协作平台。"
    }
  ]
}
```

## Type mapping

Normalize common labels:

| Input labels | Canonical type |
|---|---|
| 单选, 单项选择, single, radio | `single` |
| 多选, 多项选择, multiple, checkbox | `multiple` |
| 判断, 判断题, true/false, boolean | `boolean` |

Judgment questions should be normalized to options:

```json
[
  { "key": "正确", "text": "正确", "display": "正确" },
  { "key": "错误", "text": "错误", "display": "错误" }
]
```

## Recommended Markdown format

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

## 2. 快捷表情有哪些好处？
题型：多选
分值：10

A. 简单快捷
B. 不会触发群内其他人的通知
C. 不会刷屏
D. 可以让沟通更加严肃正式

答案：A,B,C
解析：快捷表情适合低打扰表达。
```

## Recommended CSV / Excel columns

| Column | Required | Notes |
|---|---|---|
| 题号 | yes | integer, unique |
| 题型 | yes | 单选 / 多选 / 判断 |
| 题目 | yes | question text |
| A | for choice questions | option text without prefix, or with prefix if already formatted |
| B | for choice questions | option text |
| C | optional | option text |
| D | optional | option text |
| E+ | optional | additional options allowed |
| 正确答案 | yes | keys like `D` or `A,B,C`, or full option display text |
| 分值 | yes | positive number |
| 解析 | no | explanation text |

Example row:

```csv
题号,题型,题目,A,B,C,D,正确答案,分值,解析
1,单选,飞书是什么？,聊天软件,视频会议工具,在线文档平台,一站式企业协作平台,D,10,飞书是一站式企业协作平台
```

## Validation checklist

Before creating anything, validate:

1. At least one question exists.
2. Question indexes are unique and ordered.
3. Every question has type, title, score, answer.
4. Scores are positive numbers.
5. Correct answer keys exist in options.
6. Single and boolean questions have exactly one answer.
7. Multiple questions have at least two possible options and at least one correct answer.
8. Passing score is <= total score.
9. Duplicate question titles are flagged for user confirmation.
10. Option text is normalized and trimmed; preserve intentional content but remove accidental leading/trailing spaces.

If validation fails, stop and ask the user to fix or approve a correction. Do not create partial Base resources from an invalid question bank.
