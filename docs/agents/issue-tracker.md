# Issue tracker：本地 Markdown

本项目的问题跟踪以 `.scratch/` 下的本地 Markdown 文件为准。技能要求「发布到 issue tracker」时，写入本地文件，不据此创建 GitHub Issue。

## 目录与文件

- 每项功能一个目录：`.scratch/<feature-slug>/`。
- 工作规格入口：`.scratch/<feature-slug>/spec.md`。
- 实现问题一题一文件：`.scratch/<feature-slug>/issues/<NN>-<slug>.md`，从 `01` 编号；不合并为一份总工单。
- 产品设计的正式文本保留在 `docs/superpowers/specs/`。已有正式设计时，`spec.md` 只记录链接及本项工作的边界，不复制出第二份权威设计。
- UI 设计源及版本快照遵循根目录 [AGENTS.md](../../AGENTS.md)，本地问题文件不能替代 Pencil 设计源。

## 问题字段与生命周期

每个问题在顶部记录：

- `Status:`：`open`、`claimed` 或 `resolved`，表示执行状态。
- `Triage:`：使用 [Triage labels](triage-labels.md) 中的一个规范标签，表示分诊结论；不与执行状态混用。
- `Type:`：需要区分问题类型时，使用 `research`、`prototype`、`grilling` 或 `task`。
- `Blocked by:`：有依赖时列出同一工作目录内的工单编号，如 `01, 02`；无依赖时省略。

处理流程：

1. 创建工单，写明问题、预期结果及验收标准，初始执行状态为 `open`。
2. 根据已确认的任务范围分诊。准备实施的工单须具有明确的验收条件；分诊标签不替代用户批准或其他执行门禁。
3. 在开始工单工作前，将执行状态改为 `claimed` 并保存。
4. 评论和讨论追加到 `## Comments`，不覆盖原问题和历史证据。
5. 完成后在 `## Answer` 记录结论、验证证据和剩余限制；满足验收条件才改为 `resolved`。

「读取相关 ticket」表示读取指定路径对应的本地文件。只给出编号且存在多个同号工单时，先确定所属功能，不猜测匹配。

## 有探索地图时

需要按问题逐步探索时使用以下约定；不为普通小任务强制创建地图：

- 地图：`.scratch/<effort>/map.md`，包含 Notes、Decisions-so-far、Fog。
- 子工单：`.scratch/<effort>/issues/<NN>-<slug>.md`，沿用上述字段。
- 依赖全部为 `resolved` 后，工单才解除阻塞。
- 选择下一题时，只考虑 `open`、未阻塞且已获准执行的工单，按编号从小到大选择。
- 解决子工单后，将简短结论及工单链接追加到地图的 Decisions-so-far。

## 公开仓库边界

本地记录不自动等于公开交付。提交 `.scratch/` 文件前逐项检查内容；凭据、个人信息、内部链接和未获准公开的材料不得随工单推送到公开仓库。
