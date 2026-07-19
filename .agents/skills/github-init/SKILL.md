---
name: github-init
description: >-
  初始化本地项目并发布到 GitHub。Use when the user says github-init,
  初始化 GitHub 仓库, 初始化项目并推送 GitHub, 创建远程仓库, publish local project
  to GitHub, GitHub init, or asks to turn a local project into a GitHub remote
  repository. The master agent gathers the required repository choices and
  delegates all git and gh command execution to the github-initializer agent.
disable-model-invocation: true
---

# GitHub Init

## 概览

初始化本地项目并推送到 GitHub。master agent 只负责读取规则、确认用户选择、委托执行和转述结果；所有 `git` 与 `gh` 命令都交给 `agents/github-initializer.md`。

固定规则：

- GitHub 仓库默认 `private`。
- 仓库名默认当前目录名，允许用户自定义。
- 初始提交信息固定为 `chore: initial commit`。
- 本地提交使用 `git commit`；GitHub 认证、仓库创建和远程校验使用 `gh`。
- 不创建或依赖 `agents/openai.yaml`。

## Master Agent 职责

只作为入口协调者行动。

要做：

- 接收或询问 `project_path`，默认使用当前工作目录。
- 读取目标项目的 `AGENTS.md`，并遵守其中规则。
- 如果目标项目没有 `AGENTS.md`，停止并要求用户先建立项目规则。
- 询问仓库可见性；默认 `private`，只接受 `private` 或 `public`。
- 询问仓库名称；默认使用 `project_path` 的目录名。
- 固定传入 `commit_message: "chore: initial commit"`。
- 读取 `agents/github-initializer.md`，并把 `project_path`、`repo_name`、`visibility`、`commit_message` 委托给 github-initializer subagent。
- 要求 github-initializer 按 `agents/github-initializer.md` 中的协议只返回严格 JSON。
- 收到 JSON 后用中文转述结果；如果返回内容不是可解析 JSON，按失败处理。

不要做：

- 不要运行任何 `git` 或 `gh` 命令，包括 `git status`、`git init`、`git add`、`git commit`、`git remote`、`git push`、`gh auth status`、`gh repo create` 或 `gh repo view`。
- 不要自行判断工作区是否干净、是否已有 commit、是否存在 remote、是否存在 secret 风险。
- 不要生成或修改 commit message。
- 不要自动把仓库设为 `public`；只有用户选择 `public` 才传入 `public`。

如果缺少 subagent/worker 能力，停止并说明无法在“master agent 不执行 git/gh 命令”的约束下完成初始化；不要由 master agent 接管命令执行。

## 用户参与点

在委托执行前必须确认两个选择：

- `visibility`：询问用户仓库是 `private` 还是 `public`。如果用户没有选择，使用 `private`。
- `repo_name`：询问用户仓库名称。默认值是当前目录名，用户可以改名。

如果用户已经在原始请求中明确给出这两个值，不要重复询问；直接使用用户给出的值。

## 委托 Prompt

读取 `agents/github-initializer.md` 后，使用以下 prompt 形状委托：

```text
Use the github-initializer instructions at:
<skill_path>/agents/github-initializer.md

Initialize the local project and publish it to GitHub. Return strict JSON only.

Inputs:
- project_path: <absolute project path>
- repo_name: <GitHub repository name>
- visibility: <private|public>
- commit_message: chore: initial commit

Constraints:
- The master agent will not run git or gh commands.
- You are responsible for repository inspection, local initialization, fixed initial commit, GitHub repository creation, remote setup, push, and result packaging.
- Return only the JSON contract defined in github-initializer.md. Do not return Markdown or explanatory text.
```

## Reference files

`agents/` 目录包含 specialized subagent 指令。只在需要 spawn 对应 subagent 时读取：

- `agents/github-initializer.md`：检查 `git`/`gh`、初始化本地仓库、创建固定初始提交、创建 GitHub 远程仓库、显式 push，并按固定 JSON 协议返回结果。
