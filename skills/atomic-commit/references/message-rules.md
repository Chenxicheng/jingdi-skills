# 按需 Commit Message 规则

## Issue association

- 用户明确表达“关联”、“涉及”或未声明完成 Issue：`Refs #123`。
- 用户明确表达“关闭”、“修复”或“解决” Issue：`Closes #123`。
- 同仓库使用 `#123`；跨仓库 Issue 或完整 URL 规范化为 `owner/repo#123`。
- 多个 Issue 为每个 Issue 重复完整 footer，可以在同一 message 中混合 `Refs` 与 `Closes`。
- 同一 Issue 贯穿多个 commit 时，前置 commit 使用 `Refs`，仅完成该 Issue 的最后一个 commit 使用
  `Closes`。

Issue 编号或 URL 不明确时省略 Issue footer。footer 的自然语言使用英文。

## Breaking change

变更明确破坏兼容性时：

- 在 `type!:` 或 `type(scope)!:` 中加入 `!`。
- 添加 footer：`BREAKING CHANGE: <description>`。
- 用英文描述用户可见影响或迁移要求。

可能破坏兼容性但影响不明时阻断提交，通过 Result Contract 请求缺失信息。

## Body

body 仅用于补充用户影响、风险、迁移步骤或必要理由，使用英文。
