# 仓库工作流

## 项目结构

- `skills/<skill-name>/`：每个 Skill 一个目录，目录名使用小写 hyphen-case。
- `skills/<skill-name>/SKILL.md`：必需入口文件。
- `skills/<skill-name>/agents/`：可选的任务型 subagent 指令。
- `skills/<skill-name>/references/`：可选的按需参考资料。
- `skills/<skill-name>/scripts/`：可选的确定性辅助脚本。
- `skills/<skill-name>/assets/`：可选的模板、图片和输出资源。
- `skills/` 是唯一源码目录；仓库内自有 Skill 可通过相对软链接暴露到
  `.agents/skills/` 和 `.claude/skills/`，需要生成副本时才使用 `skills` CLI 更新。
- `tests/`、`docs/` 和 `scripts/` 分别存放自动化验证、仓库文档和仓库级维护脚本。
- `docs/plans/`：存放 Plan mode 或执行计划产生的仓库级计划文档，命名遵循 `YYYYMMDD-<feature-name>-v{num}.md`。

根目录只放 `README.md`、`LICENSE`、`Makefile`、`package.json` 和配置文件等标准项目文件，不放零散产出物。

## 工具链

当前没有统一的 build、test 或 lint 命令。引入工具链时，在本文件记录稳定入口，优先使用：

- `make test` 或 `npm test`
- `make lint` 或 `npm run lint`
- `make build` 或 `npm run build`
- `make dev` 或 `npm run dev`

## 实现规范

- 遵循所选语言和框架的惯例，保持实现简单、清晰、可验证。
- 文件名、标识符、命令和代码注释使用英文。
- 使用描述性名称，少用缩写；模块名遵循生态的 hyphen-case 或 snake_case 约定。
- 已有 formatter 或 linter 时只使用项目配置，不引入竞争工具。

## 测试与同步

- 用户可见行为、自动化或可复用逻辑必须测试正常路径、边界和失败路径。
- 测试布局尽量镜像源码；除非标记为集成测试，否则不依赖外部网络服务。
- 修改后运行适用的 test、lint 和 build，并完成 code review。
- 新增或修改 Skill 后验证发现入口：软链接使用 `readlink` 和内容比对；生成副本运行
  `npx skills add . --skill <skill-name> --agent codex -y`，同步 `.agents/skills/` 和
  `skills-lock.json`。

## Git 与安全

- commit message 使用简洁英文描述变更意图。
- Pull Request 说明变更摘要、原因和验证结果；行为或体验变化附截图或终端输出，并链接相关 issue 或后续任务。
- 本地配置放入已忽略文件；需要示例时提供不含凭据的 `.env.example`。
