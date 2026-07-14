# 仓库规范

## 核心约束

- 本仓库是个人 Skills 试炼仓库，用于创建、试用、评估和迭代 Skill。
- 文档及其他产出物默认使用中文；技术术语、代码、命令、文件名、标识符和代码注释使用英文。
- 每个 Skill 独立且自包含。除非作者明确引用，不读取、继承或依赖其他 Skill。
- 密钥、token、私钥和机器专属凭据不得进入仓库。

## 条件式引用

Codex 不会因 Markdown 链接自动加载文件。命中下列条件时，必须先完整读取对应文档，再开始工作：

- 创建、修改、评估或审查 Skill：读取 [Skill 编写规范](docs/skill-authoring.md)。
- 实现或修改仓库内容，或者执行测试、同步、Commit 和 Pull Request：读取 [仓库工作流](docs/repository-workflow.md)。

引用文件只保存对应领域的规则；根 `AGENTS.md` 只保存始终生效的约束和加载条件。

## 计划文档

- Plan mode 或执行计划产生的仓库级计划文档必须存放在 `docs/plans/`。
- 文件名必须使用 `YYYYMMDD-<feature-name>-v{num}.md`，例如 `20260713-skill-sync-v1.md`。
- `<feature-name>` 使用英文 hyphen-case，表达计划对象或功能名。
- 同一 feature 的新计划禁止更新原有计划文件；必须创建下一个版本号的新文件，例如从 `v1` 升级为 `v2`。
