---
name: atomic-commit
description: >-
  创建 atomic commit。Use when the user asks to create git commits from current
  changes or specified files, or to split work into one or more atomic commits;
  also use for 提交 or 帮我提交.
---

# Atomic Commit

## 概览

创建一个或多个 atomic commits，并把所有提交相关工作交给 committer subagent。master agent 只负责触发、收集最低输入、读取 `agents/committer.md`、委托、校验 JSON、转述结果。

## Master Agent 职责

只作为入口协调者行动。

要做：

- 接收或询问 `project_path`。
- 解析当前 skill 目录的真实绝对路径，作为 `skill_path` 传给 committer；无法解析时停止，不 spawn committer。
- 保留用户原始提交意图，作为 `intent` 传给 committer。
- 如果用户明确给了文件列表，把它作为可选 `commit_files` 传给 committer。
- 读取 `agents/committer.md`，并把 `skill_path`、`project_path`、可选 `commit_files`、`intent` 委托给 committer subagent。
- 要求 committer 按 `agents/committer.md` 中的协议只返回严格 JSON。
- 收到 JSON 后解析并校验字段和类型；非 JSON、缺字段、字段类型错误、未替换占位符或自然语言前后缀都按失败处理。

不要做：

- 运行任何 bash git 命令，包括 `git status`、`git diff`、`git add`、`git commit`、`git restore`、`git reset`、`git push` 或 `git config`。
- 检查 git 状态、推断相关文件、判断原子性、拆分 commit group、生成 commit message 或执行 commit。
- 要求 subagent 提交 "everything"；没有文件列表时，传入用户原始意图，让 committer 自行识别相关 atomic commit 候选。
- 在提交后 push。

如果缺少 `project_path`，向用户询问。`commit_files` 可以缺省；缺省时由 committer 从 git 状态和用户意图中识别候选。

如果当前环境没有可用的 subagent/worker 能力，停止并向用户说明无法在“不由 master agent 执行 git 操作”的约束下完成提交；不要由 master agent 接管 git 命令。

## 委托 Prompt

读取 `agents/committer.md` 后，使用以下 prompt 形状委托：

```text
Use the committer instructions at:
<skill_path>/agents/committer.md

Create one or more atomic commits and return strict JSON only.

Inputs:
- skill_path: <absolute path to this skill folder>
- project_path: <absolute project path>
- commit_files: <optional explicit list of files>
- intent: <user's original commit request>

Constraints:
- The master agent will not run git commands.
- You are responsible for related-file detection, atomicity decisions, git inspection, staging, committing one or more atomic commits, and result packaging.
- Return only valid JSON matching the contract defined in committer.md. Do not return Markdown or explanatory text.
```

## Reference files

`agents/` 目录包含 specialized subagent 指令。只在需要 spawn 对应 subagent 时读取：

- `agents/committer.md`：识别相关变更、拆分一个或多个 atomic commits、检查 git 状态、保护 staged work、生成 commit message、执行 commit，并按固定 JSON 协议返回结果。

`references/` 目录包含 committer 按需读取的详细规则：

- `references/message-rules.md`：只有当 diff 或 `intent` 暗示 breaking change、issue reference、body/footer 时读取。
