---
name: atomic-commit
description: 创建本地原子化 Git commit：提交当前或指定范围的变更，或将可按完整文件分组的混合变更拆分为多个 commit。
---

# 原子化提交

## Coordinator 流程

1. 从用户请求或 workspace metadata 解析绝对路径 `project_path`；两者均无法确定时才询问。
   解析当前 Skill 目录为 `skill_path`，不读取 Commit Guide。
2. 选择 `mode`：完整文件列表和已检查的改动都在当前上下文中时用 `direct`，按统一能力预分组
   （避免过多 commit）并传入 `commit_groups`；
   否则用 `inspect`，仅在用户指定路径时传入最大范围 `commit_files`。
   将相关目的、范围、Issue 线索与关闭语义整理为 `intent`。
3. 创建不继承对话历史的新 Subagent，不指定 model。仅发送 Delegation Envelope 并等待。
4. 核对返回的 JSON 结构并转述结果；格式错误时报告异常，不再运行 Git。

Subagent 独占仓库检查、分组、commit message、Git 写入与结果记录；不可用时停止。

## Delegation Envelope

```text
作为本次请求唯一的 Subagent。

运行任何 Git 命令前，先读取并遵循：
<skill_path>/references/commit-guide.md

输入：
- project_path: <项目绝对路径>
- mode: direct | inspect
- intent: <本次提交的相关上下文>
- commit_groups: [{"purpose": "<该组目的>", "files": ["<路径>"]}]（direct 必传，为完整预分组）
- commit_files: ["<路径>"]（inspect 可选，为最大允许范围）

按 `mode` 独立完成检查与分组、commit 创建和结果记录。只返回一个 JSON 对象，不添加 Markdown 或前后文本。

Result Contract

成功：
{"ok":true,"data":{"commits":[{"commitId":"<id>","commitMessage":"<message>","files":["<repo-relative path>"]}],"ignoredFiles":["<repo-relative path>"]}}

失败：
{"ok":false,"data":null,"error":{"message":"<reason>","files":[],"nextStep":"<smallest required action>"}}
失败时若仓库已解析，data 写入 repoName、branch、commits、ignoredFiles；部分成功时 commits 写入已创建项。
```
