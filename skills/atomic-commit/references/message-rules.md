# Message Rules

只在 diff 或 `intent` 暗示 breaking change、issue reference、body/footer 时读取本文件。普通 one-line Conventional Commit 不需要读取。

## Breaking Changes

如果候选 diff 或 `intent` 明确表示破坏性变更：

- subject 使用 `type!:` 或 `type(scope)!:`。
- footer 必须包含 `BREAKING CHANGE: <description>`。
- `<description>` 必须说明用户可见影响或迁移要求。

如果只能判断“可能 breaking”，但影响说明不清楚，返回失败 JSON，请用户补充 breaking impact。不要发明破坏性影响。

## Issue Footers

只使用用户 `intent` 或候选 diff 中可见的 issue reference：

- 修复并关闭 issue：`Closes #123`
- 相关但不关闭 issue：`Refs #456`

不要猜 issue，不要从无关文本生成 issue footer。

## Body And Footer

body 只用于说明用户影响、风险、迁移说明或变更原因。普通 one-line commit 不添加 body。

footer 只用于 breaking change 或 issue reference。没有明确来源时，不添加 footer。
