# lark-exam-base-builder

一个用于 **Claude Code** 的飞书 / Lark 多维表格考试系统构建 Skill：根据题库自动创建飞书 Base 考试管理微系统。

它可以把 Markdown、CSV、Excel 风格表格或直接粘贴的题库，转换成一套可用的飞书多维表格考试系统，包括：

- 标准答案与分值表；
- 答卷汇总表；
- 答题表单；
- 每题自动判分公式；
- 总分与是否及格计算；
- 答卷汇总视图和不及格名单视图；
- 可选的考试看板。

> English summary: A Claude Code skill for building Feishu/Lark Base exam management systems from question banks.

## 会创建什么

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

## 支持的题型

| 题型 | 规范值 | 表单字段 | 判分方式 |
|---|---|---|---|
| 单选 | `single` | 单选 select | 精确匹配 |
| 多选 | `multiple` | 多选 select | 支持时按选项集合完全一致判分 |
| 判断 | `boolean` | 单选 select | `正确` / `错误` 精确匹配 |

填空题、问答题等暂不作为 v1 自动判分题型。遇到这类题目时，建议跳过、转为人工判分，或后续改用规范化的“题库表 + 答卷表 + 答题明细表”结构。

## 使用前提

- 已安装 Claude Code；
- 已安装 `lark-cli`；
- 已在 `lark-cli` 中登录飞书 / Lark 账号；
- 当前账号有创建和编辑飞书多维表格资源的权限。

本 Skill 使用 `lark-cli base +...` 快捷命令，例如：

```bash
lark-cli base +base-create
lark-cli base +table-create
lark-cli base +record-batch-create
lark-cli base +field-create
lark-cli base +form-create
lark-cli base +form-questions-update
lark-cli base +dashboard-create
```

## 安装

将仓库克隆到 Claude Code 的 skills 目录：

```bash
git clone https://github.com/jonas857/lark-exam-base-builder.git ~/.claude/skills/lark-exam-base-builder
```

如有需要，重启 Claude Code 或重新加载 skills。

## 使用示例

可以这样对 Claude Code 说：

```text
根据这份题库创建飞书考试系统
```

```text
把这个 Markdown 题库变成飞书多维表格考试表单，并自动判分
```

```text
创建企业培训考试管理微系统，包含答卷汇总、不及格名单和考试看板
```

Skill 的标准流程是：

1. 将题库解析为统一的 canonical schema；
2. 校验题号、题型、选项、答案、分值和及格线；
3. 展示创建计划；
4. 在创建飞书持久化资源前请求用户确认；
5. 创建 Base、表、字段、视图、表单、公式和看板；
6. 返回创建结果、URL / ID，以及任何部分失败信息。

## 输入格式

详见：

- [`references/exam-question-format.md`](references/exam-question-format.md)
- [`references/examples/sample-questions.md`](references/examples/sample-questions.md)
- [`references/examples/sample-questions.csv`](references/examples/sample-questions.csv)

最小 Markdown 示例：

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

## 设计说明

默认采用 **混合公式模式（hybrid formula mode）**：

- `标准答案分值表` 保存规范化题库、标准答案、分值和解析，便于人工查看和维护；
- `答卷汇总表` 使用自动生成的 formula 字段直接判分。

相比 lookup 判分，这种方式在自动化生成时更稳定，尤其适合处理 select 选项同步、多选题和字段命名问题。

## 重要实现细节

参考文档中记录了真实端到端测试得到的操作经验：

- 只使用 `lark-cli base +...` 快捷命令；
- 除非用户明确要求 bot 身份，否则 Base 命令默认加 `--as user`；
- 大型 `--json` / `--fields` 参数优先使用当前目录相对路径，例如 `@./file.json`；
- `+form-questions-update --questions` 需要传入行内 JSON 字符串，不支持 `@file`；
- Windows / Git Bash 下更新表单中文标题时，建议用 `ensure_ascii=True` 生成 JSON，避免中文乱码；
- 创建公式字段时可能需要 `--i-have-read-guide`；
- 答卷表字段名可以短，但表单题目标题必须更新为完整题干；
- Dashboard 的 `group_by.sort` 应同时包含 `type` 和 `order`。

## 文档索引

- [`SKILL.md`](SKILL.md) — Claude Code Skill 入口和主流程。
- [`references/base-schema.md`](references/base-schema.md) — 生成的 Base 结构设计。
- [`references/exam-question-format.md`](references/exam-question-format.md) — 支持的题库格式与 canonical JSON。
- [`references/scoring-formulas.md`](references/scoring-formulas.md) — 判分公式策略和已验证公式格式。
- [`references/lark-cli-command-flow.md`](references/lark-cli-command-flow.md) — CLI 命令顺序和注意事项。
- [`references/troubleshooting.md`](references/troubleshooting.md) — 常见问题与恢复方案。

## 安全说明

本 Skill 会创建飞书 / Lark 持久化资源，因此应始终遵守：

- 创建资源前必须让用户确认；
- 删除、隐藏或覆盖已有资源前必须确认具体目标；
- 如出现部分失败，需要如实报告已经创建了什么、还有什么需要手动完成；
- 不要公开飞书 token、私有 Base 链接、真实答卷、员工或学生数据。

## License

MIT. See [`LICENSE`](LICENSE).
