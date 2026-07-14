---
name: atomic-commit
description: >-
  创建原子化 Git 提交并把 Git 作业委派给 committer subagent。用于 Git
  仓库上下文中用户明确要求立即提交当前工作或指定范围、把混合变更拆分并创建
  多个 commit、仅说“提交”，或项目规范要求委派已批准的提交操作。不要用于
  amend、初始化仓库、initial commit、push、分析拆分方案、只整理 staging，或
  其他不创建 commit 的 Git 作业。
---

# 原子化提交

创建一个或多个原子化提交。每个 commit 只表达一个单一、可审查的目的；完成
该目的所需的实现、测试、文档、配置、迁移和生成产物合并为同一个 commit。

## 主控职责

只做协调入口。

1. 解析 `project_path`；缺失时才询问用户。
2. 解析当前 skill 目录的绝对路径为 `skill_path`；无法解析时停止。
3. 保留用户原始请求为 `intent`。
4. 用户明确指定文件或目录范围时，作为可选 `commit_files` 传入；否则省略该
   字段，让 committer 根据 `intent` 识别候选组。
5. 创建 committer subagent，并要求它在任何 Git 命令前读取
   `<skill_path>/agents/committer.md`。
6. 按 `agents/committer.md` 的完整字段契约校验严格 JSON，再向用户转述结果。

不要运行 Git 命令、检查 status 或 diff、选择文件、拆分候选组、编写
commit message、执行 commit、amend 或 push。没有 subagent/worker 能力时，
停止并说明本 skill 需要委派执行 Git 作业。

## Subagent 提示词

```text
先读取并遵循以下 committer 指令：
<skill_path>/agents/committer.md

创建一个或多个原子化提交，并且只返回严格 JSON。

输入：
- skill_path: <当前 skill 目录的绝对路径>
- project_path: <项目绝对路径>
- commit_files: <可选的明确文件或目录范围；缺失时省略>
- intent: <用户原始提交请求>

约束：
- master agent 不运行 Git 命令。
- 你负责 Git 检查、相关文件识别、原子化分组、staging、提交、校验和结果封装。
- 只返回符合 committer.md 的有效 JSON。不要返回 Markdown 或说明文字。
```

## 资源

- `agents/committer.md`：必读的 subagent 操作指引，覆盖低 token Git 检查、
  原子化候选组识别、index 保护、commit 创建和 JSON 结果封装。
- `references/message-rules.md`：只有候选组需要 breaking change、issue
  footer、body 或其他高级 commit message 细节时读取。
