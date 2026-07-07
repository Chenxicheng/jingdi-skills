# 仓库规范

## 项目结构与模块组织

本仓库用于存放 Codex 兼容的 skill 包。添加实现文件前，先遵守以下结构：

- `skills/<skill-name>/`：每个 skill 一个目录，目录名使用小写 hyphen-case。
- `skills/<skill-name>/SKILL.md`：必需文件，存放 skill 指令和触发元数据。
- `skills/<skill-name>/agents/`：可选目录，存放任务型 subagent 指令，例如 `committer.md`。
- `skills/<skill-name>/references/`：可选目录，存放按需加载的详细参考资料。
- `skills/<skill-name>/scripts/`：可选目录，存放 skill 使用的确定性辅助脚本。
- `skills/<skill-name>/assets/`：可选目录，存放模板、图片或其他输出资产。
- `tests/`：添加可复用脚本或生成产物时，用于自动化验证。
- `docs/`：需要仓库级文档时使用。
- `scripts/`：需要仓库级自动化或维护脚本时使用。

不要在根目录添加零散文件，标准项目文件除外，例如 `README.md`、`LICENSE`、`Makefile`、`package.json` 或配置文件。

## 构建、测试与开发命令

当前没有构建或测试命令。引入工具链时，必须在同一次变更中记录标准命令，并优先提供稳定封装：

- `make test` 或 `npm test`：运行完整自动化测试。
- `make lint` 或 `npm run lint`：运行格式化和静态检查。
- `make build` 或 `npm run build`：生成可分发产物。
- `make dev` 或 `npm run dev`：启动本地开发。

只要某个命令是贡献流程必需的，就必须在引入它的同一次变更中写进本文件。

## 编码风格与命名约定

按每个模块所选语言和框架的惯例编写。保持代码简单、清晰、可验证。

- 文件名、标识符、命令和代码注释使用英文。
- 优先使用描述性名称，少用缩写。
- 模块名使用小写，并按生态惯例选择 hyphen 或 underscore。
- 一旦项目配置了 formatter 或 linter，就使用项目配置；不要混用竞争性的风格工具。

## 测试规范

当添加用户可见行为、自动化逻辑或可复用逻辑时，补充测试。测试布局尽量镜像源码布局，例如 `src/parser.ts` 对应 `tests/parser.test.ts`。

覆盖正常路径、边界情况和失败路径。除非明确标记为集成测试，否则避免依赖外部网络服务。

## Commit 与 Pull Request 规范

本仓库还没有提交历史，commit message 使用简洁英文描述变更意图，例如 `Add initial skill structure`。

Pull request 应包含：

- 变更摘要，以及为什么需要它。
- 已执行的验证，例如 `make test` 或 `npm test`。
- 涉及行为或体验变化时，附截图或终端输出。
- 相关 issue 或后续任务链接。

## 安全与配置

不要提交密钥、token、私钥或机器专属凭据。把本地配置放在已忽略文件中，并在需要时提供安全示例，例如 `.env.example`。
