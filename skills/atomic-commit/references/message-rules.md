# 高级提交信息规则

只在 diff 或 `intent` 暗示破坏性变更、issue 引用、正文或 footer 时读取本文件。
普通单行 Conventional Commit 不需要读取。

## 破坏性变更

如果候选 diff 或 `intent` 明确表示破坏性变更：

- subject 使用 `type!:` 或 `type(scope)!:`。
- footer 必须包含 `BREAKING CHANGE: <description>`。
- `<description>` 使用英文说明用户可见影响或迁移要求。

如果只能判断“可能破坏兼容”，但影响说明不清楚，返回失败 JSON，请用户补充影响说明。不要发明破坏性影响。

## Issue Footer

只使用用户 `intent` 或候选 diff 中可见的 issue 引用：

- 修复并关闭 issue：`Closes #123`
- 相关但不关闭 issue：`Refs #456`

不要猜 issue，不要从无关文本生成 issue footer。

## 正文与 Footer

body 使用英文，只说明用户影响、风险、迁移要求或变更原因。普通单行 commit 不添加 body。

footer 使用英文，只用于破坏性变更或 issue 引用。没有明确来源时，不添加 footer。
