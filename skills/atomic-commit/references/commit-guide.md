# Subagent 执行指南

作为本次请求唯一的 Subagent，独占全部 Git 操作。从委派输入接收 `project_path`、`intent`、
可选 `commit_files` 和 Result Contract。

## 边界

- 每个 commit 只表达一个可审查目的，并包含该目的需要的实现、测试、文档、配置、迁移与生成产物。
- `commit_files` 是最大允许范围；完整 commit 需要范围外路径时阻断。
- 需要 hunk 级拆分、单文件混合多个目的、依赖或顺序不明时阻断，不运行交互式 Git 命令。
- 敏感文件、密钥、token、密码、私钥、`.env` 或机器本地配置始终阻断。
- 二进制或生成产物仅在用户明确纳入，或与同组 source/producer 的关系清晰时提交。
- 不 push、amend、rollback、绕过 Hook、修改 Git 配置或丢弃 worktree 变更。
- 默认只做提交级验证；test、lint 或 build 仅在 `intent` 明确要求时运行。

## 1. 解析并清点

在 `project_path` 中运行：

```bash
git rev-parse --show-toplevel
git rev-parse --verify HEAD
git branch --show-current
git status --branch --untracked-files=all
```

阻断 unborn `HEAD`、unmerged 条目或进行中的 merge、rebase、cherry-pick 与 revert。依据 `intent` 和
`commit_files` 将每个变更路径归入候选分组、无关忽略或阻断错误。`repoName` 取 repository root
目录名；detached HEAD 的 `branch` 返回 `HEAD-detached:<short-hash>`。所有候选路径使用 repo-relative
literal path。

**完成条件：** 仓库状态允许提交，且初始变更路径全部完成分类。

## 2. 审查并分组

审查每组 tracked 变更：

```bash
GIT_LITERAL_PATHSPECS=1 git diff HEAD -- "${candidate_files[@]}"
GIT_LITERAL_PATHSPECS=1 git diff --check HEAD -- "${candidate_files[@]}"
```

直接读取每组 untracked 文本文件；二进制和生成产物按边界规则判定。检查原子性、完整性、敏感信息与
依赖顺序；依赖分组在前，其余按稳定路径顺序。运行 `intent` 明确要求的额外验证。

**完成条件：** 每个候选路径恰好属于一个完整分组，所有分组通过审查并排好顺序。

## 3. 生成 message

为每组创建不超过 72 个字符的英文祈使语气 Conventional Commit subject：

```text
<type>(optional-scope): <imperative summary>
```

`type` 取 `feat`、`fix`、`docs`、`style`、`refactor`、`perf`、`test`、`build`、`ci`、`chore`、
`revert`。仅当 `intent` 明确给出 Issue reference/关闭语义、变更属于 breaking change，或 message 需要
补充用户影响、风险、迁移步骤或必要理由时，读取 [`message-rules.md`](./message-rules.md)。Issue 不得
从 branch 或 diff 推断。

**完成条件：** 每个分组都有一条符合其目的与按需 message 规则的独立 message。

## 4. 逐组提交

对每个分组，依次执行本节提交和第 5 节校验；当前分组校验通过后才处理下一分组。当前组包含
untracked 文件时，先仅为这些路径记录 intent-to-add：

```bash
GIT_LITERAL_PATHSPECS=1 git add -N -- "${untracked_files[@]}"
GIT_LITERAL_PATHSPECS=1 git commit --only -m "$subject" -- "${candidate_files[@]}"
```

body 和 footer 使用独立 `-m` 参数。`git add -N` 后、commit 创建前发生失败时，只清理本次新增的
intent-to-add 路径：

```bash
GIT_LITERAL_PATHSPECS=1 git reset -q HEAD -- "${untracked_files[@]}"
```

**完成条件：** 当前分组创建一个 commit；失败时立即停止后续分组，保留已完成 commit。

## 5. 提交后校验与返回

当前组 commit 创建后立即运行：

```bash
git diff-tree --no-commit-id --name-only -r --no-renames HEAD
git show -s --format='%H%n%B' HEAD
GIT_LITERAL_PATHSPECS=1 git status --short --untracked-files=all -- "${candidate_files[@]}"
```

要求实际文件集合与候选分组完全相等、message 与预期相等，且候选范围无残留变更。只记录验证通过的
commit。任一检查失败时保留已创建 commit，在 `error.message` 中写明完整 commit hash 和失败检查，
在 `nextStep` 中给出最小处理动作，并将未验证路径写入 `error.files`。

严格按委派输入的 Result Contract 返回 JSON。`ignoredFiles` 仅来自初始清点的无关路径；部分成功时返回
全部已验证 commit 和未提交的错误路径。JSON 之外不返回其他文本。

**完成条件：** 返回值符合 Result Contract，且每个初始变更路径只出现在 committed、ignored 或 error
其中一类。
