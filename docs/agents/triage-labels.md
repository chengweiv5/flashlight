# Triage labels

本项目使用以下五个规范分诊标签，记录在本地问题文件的 `Triage:` 字段中。

| 规范标签 | 本项目标签 | 含义 |
| --- | --- | --- |
| `needs-triage` | `needs-triage` | 需要维护者评估问题及处理方式 |
| `needs-info` | `needs-info` | 信息不足，等待补充资料或澄清 |
| `ready-for-agent` | `ready-for-agent` | 需求和验收条件明确，适合 Agent 执行 |
| `ready-for-human` | `ready-for-human` | 需要人类实施或完成关键操作 |
| `wontfix` | `wontfix` | 已决定不处理，须记录原因 |

- 技能提及分诊角色时，使用表中的精确字符串，不另造同义标签。
- 一个问题使用一个当前分诊标签；变更原因追加到 `## Comments`。
- 分诊与执行状态分开：`Triage:` 不使用 `open`、`claimed`、`resolved`；这些属于 `Status:`。
- `ready-for-agent` 只表示工单准备充分，不授予发布、破坏性操作或绕过设计审批的权限。
- `wontfix` 不是「已经实现」；记录不处理的决定后，才可将执行状态设为 `resolved`。
- 具体文件路径、字段和处理流程见 [Issue tracker](issue-tracker.md)。
