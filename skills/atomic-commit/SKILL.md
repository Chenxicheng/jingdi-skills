---
name: atomic-commit
description: >-
  提交已有 Git 仓库的当前或指定范围变更：创建新的本地原子化 commit，或将可按完整文件分组的
  混合变更拆分为多个 commit。不用于 amend、initial commit、push 或仅 staging。
---

# 原子化提交

## Coordinator 流程

1. 从用户请求或 workspace metadata 解析绝对路径 `project_path`；两者均无法确定时才询问。
   解析当前 Skill 目录为 `skill_path`，不读取 Commit Guide。
2. 将完整提交请求作为 `intent`，包含请求上下文中明确的 Issue reference 与关闭语义；
   不从 branch 或 diff 推断 Issue。用户明确指定路径时，将其作为最大范围 `commit_files`。
3. 创建不继承对话历史的新 Subagent，不指定 model。仅发送委派输入并等待。
4. 按 Result Contract 校验 JSON。不合格时，只允许同一 Subagent 在不运行 Git 的前提下修正一次；
   仍不合格则报告阻塞，否则转述结果。

Subagent 独占仓库检查、分组、commit message、Git 写入与校验。Coordinator 仅执行上述
流程；Subagent 不可用时停止。

## 委派输入

```text
作为本次请求唯一的 Subagent。

运行任何 Git 命令前，先读取并遵循：
<skill_path>/references/commit-guide.md

输入：
- project_path: <项目绝对路径>
- intent: <用户完整请求>
- commit_files: <可选的最大允许范围；缺失时省略>

独立完成仓库检查、分组、commit 创建和提交后校验。只返回一个 JSON 对象，不添加 Markdown 或前后文本。

结果契约

成功：
{"ok":true,"data":{"commits":[{"commitId":"<id>","commitMessage":"<message>","files":["<repo-relative path>"]}],"ignoredFiles":["<repo-relative path>"]}}

仓库解析后的失败：
{"ok":false,"data":{"repoName":"<name>","branch":"<branch>","commits":[],"ignoredFiles":[]},"error":{"message":"<reason>","files":["<repo-relative path>"],"nextStep":"<smallest required action>"}}

仓库解析前的失败：
{"ok":false,"data":null,"error":{"message":"<reason>","files":[],"nextStep":"<smallest required action>"}}

部分成功时，将已创建且验证通过的 commit 写入失败结果的 data.commits。
```

## 结果校验

- 只接受上述三种结构，不得增加顶层字段。
- 字符串必须非空，`files` 必须使用 repo-relative 路径；`ignoredFiles` 和失败结果的
  `error.files` 可以为空。
- 成功结果至少包含一个 commit；没有变更时返回失败。
- `commits[].files`、`ignoredFiles`、`error.files` 各自无重复且彼此互斥。
- 原样转述 `error.message` 和 `nextStep`，不得重新解释仓库事实。
