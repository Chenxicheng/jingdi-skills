---
name: atomic-commit
description: >-
  创建本地原子化 Git commit：提交当前或指定范围的变更，或将可按完整文件分组的混合变更
  拆分为多个 commit；仓库须已有 HEAD。
---

# 原子化提交

## Coordinator 流程

1. 从用户请求或 workspace metadata 解析绝对路径 `project_path`；两者均无法确定时才询问。
   解析当前 Skill 目录为 `skill_path`，不读取 Commit Guide。
2. 将完整提交请求和已知的相关上下文作为 `intent`，保留变更目的、范围、Issue 线索及
   关闭语义；未知信息留空。用户明确指定路径时，原样传入 `commit_files` 作为最大范围。
3. 创建不继承对话历史的新 Subagent，不指定 model。仅发送委派输入并等待。
4. 核对返回的 JSON 结构并转述结果；格式错误时报告异常，不再运行 Git。

Subagent 独占仓库检查、分组、commit message、Git 写入与结果记录。Coordinator 仅执行上述
流程；Subagent 不可用时停止。

## 委派输入

```text
作为本次请求唯一的 Subagent。

运行任何 Git 命令前，先读取并遵循：
<skill_path>/references/commit-guide.md

输入：
- project_path: <项目绝对路径>
- intent: <用户完整请求及相关上下文>
- commit_files: ["<用户指定路径>"]（可选最大范围；缺失时省略）

独立完成仓库检查、分组、commit 创建和结果记录。只返回一个 JSON 对象，不添加 Markdown 或前后文本。

结果契约

成功：
{"ok":true,"data":{"commits":[{"commitId":"<id>","commitMessage":"<message>","files":["<repo-relative path>"]}],"ignoredFiles":["<repo-relative path>"]}}

仓库解析后的失败：
{"ok":false,"data":{"repoName":"<name>","branch":"<branch>","commits":[],"ignoredFiles":[]},"error":{"message":"<reason>","files":["<repo-relative path>"],"nextStep":"<smallest required action>"}}

仓库解析前的失败：
{"ok":false,"data":null,"error":{"message":"<reason>","files":[],"nextStep":"<smallest required action>"}}

部分成功时，将已创建的 commit 写入失败结果的 data.commits。
```
