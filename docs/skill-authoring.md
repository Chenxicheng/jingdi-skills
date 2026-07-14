# Skill 编写规范

## 适用范围

本规范用于创建、修改、评估和审查 Skill。

除非用户指定单一平台，Skill 以 [Agent Skills Specification](https://agentskills.io/specification) 为通用格式，同时适配 Codex 和 Claude Code。仅在明确要求时添加平台专属内容，并与通用核心隔离。

## 编写依据

同时使用以下 Skill：

1. `$writing-great-skills`：主要写作准则。
2. Codex 官方 `$skill-creator`：校准 Codex 创建与验证要求。
3. Claude Code `$skill-creator`：校准测试、评估与迭代流程。

这些 Skill 只指导编写过程，不自动成为产出 Skill 的依赖。规则冲突时依次服从：用户明确要求、Agent Skills Specification、目标平台官方文档。

## 官方引用

按任务需要读取，不复述外部规范：

- 通用格式、frontmatter、命名或渐进式披露：[Agent Skills Specification](https://agentskills.io/specification)
- Claude Code 专属行为：[Claude Code Skills](https://code.claude.com/docs/en/skills)
- Codex 专属行为：[Codex Skills](https://developers.openai.com/codex/skills)
