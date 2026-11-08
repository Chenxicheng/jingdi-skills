# Subagent 执行指南

输入：`project_path`、`mode`、`intent`、`commit_files`（`inspect` 时可省略）和 Result Contract。

## 边界

- 每个 commit 只表达一个可审查目的，并包含该目的需要的实现、测试、文档、配置、迁移与生成产物。
- `direct` 的 `commit_files` 是完整提交范围；`inspect` 的 `commit_files` 是最大允许范围。
  完整分组需要范围外路径时阻断。
- 需要 hunk 级拆分、单文件混合多个目的、依赖或顺序不明时阻断，不运行交互式 Git 命令。
- 敏感文件、密钥、token、密码、私钥、`.env` 或机器本地配置始终阻断。
- 二进制或生成产物仅在用户明确纳入，或与同组 source/producer 的关系清晰时提交。
- 不 push、amend、rollback、绕过 Hook、修改 Git 配置或丢弃 worktree 变更。
- test、lint 或 build 仅在 `intent` 明确要求时运行。
- 路径与 message 均作为独立参数传给 Git；按当前执行环境安全引用动态值。

## 1. 解析并清点

在 `project_path` 中运行 `git rev-parse --show-toplevel`，以返回的仓库根目录为后续命令的工作目录：

```text
git rev-parse --verify HEAD
git branch --show-current
git status --branch --untracked-files=all
```

阻断 unborn `HEAD`、unmerged 条目或进行中的 merge、rebase、cherry-pick 与 revert。依据 `intent` 和
`commit_files` 将每个变更路径归入候选分组、无关忽略或阻断错误。将 `commit_files` 归一为仓库相对路径；
路径基准不明时阻断并报告。还原 `git status` 的文件名转义和显示引号。`repoName` 取 repository
root 目录名；detached HEAD 的 `branch` 返回 `HEAD-detached:<short-hash>`。
没有候选变更时按失败返回。

**完成条件：** 仓库状态允许提交，且初始变更路径全部完成分类。

## 2. 按模式分组

`direct`：将 `commit_files` 作为一个分组，依据 `intent` 确定目的。核对这些路径均有待提交变更，
直接进入 message 生成；无需逐文件读取 diff 或文件内容。

`inspect`：运行 `git --literal-pathspecs diff HEAD --` 审查每组 tracked 变更；直接读取每组
untracked 文本文件。检查原子性、完整性、敏感信息与依赖顺序；依赖分组在前，其余按稳定路径顺序。
二进制和生成产物按边界规则判定。

两种模式都运行 `intent` 明确要求的额外验证。

**完成条件：** `direct` 的指定路径均有变更，或 `inspect` 的每个候选路径都归入完整分组；要求的验证已通过。

## 3. 生成 message

`direct` 根据 `intent` 生成 message；`inspect` 根据已审查的变更生成 message，结合已知上下文说明目的。

为每组创建不超过 72 个字符的英文祈使语气 Conventional Commit subject：

```text
<type>(optional-scope): <imperative summary>
```

`type` 取 `feat`、`fix`、`docs`、`style`、`refactor`、`perf`、`test`、`build`、`ci`、`chore`、
`revert`。出现 Issue 线索、变更明确或可能破坏兼容性，或 message
需要补充用户影响、风险、迁移步骤或必要理由时，读取
[`message-rules.md`](./message-rules.md)。

**完成条件：** 每组都有独立 message，其内容可由本组变更和已确认的上下文支持。

## 4. 逐组提交

对每个分组，依次提交并记录第 5 节的 commit hash，再处理下一组。当前组包含
untracked 文件时，先用 `git --literal-pathspecs add -N --` 仅为这些路径记录 intent-to-add。
然后用 `git --literal-pathspecs commit --only` 创建提交：subject、每段 body 和每条 footer
分别使用一个 `-m` 参数，最后用 `--` 逐个传入本组路径。`git add -N` 后、commit 创建前发生失败时，
仅对本次新增的 intent-to-add 路径运行 `git --literal-pathspecs reset -q HEAD --` 清理标记。

**完成条件：** 当前分组创建一个 commit；失败时立即停止后续分组，保留已完成 commit。

## 5. 返回结果

每组提交成功后运行 `git rev-parse HEAD` 和 `git cat-file commit HEAD`，记录 commit hash 与
commit 对象首个空行后的实际 message。部分成功时报告已创建的 commit、失败路径和下一步。
`ignoredFiles` 仅包含初始清点中与请求无关的路径。

严格按委派输入的 Result Contract 返回 JSON。JSON 之外不返回其他文本。

**完成条件：** 已记录所有成功的 commit，并报告剩余路径的处理结果。
