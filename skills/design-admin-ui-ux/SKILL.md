---
name: design-admin-ui-ux
description: 中后台系统 UI/UX 开发与审查指引。
disable-model-invocation: true
---

# 中后台 UI/UX 开发必读指引

## 强制读取

所有中后台 UI/UX 开发开始前完整读取本文件。以下工作均属于 UI/UX 开发：

- 新增或修改页面、布局、样式、组件和响应式行为
- 新增或修改表单、表格、导航、反馈、权限展示和交互流程
- 修复影响用户可见状态、操作路径或信息表达的问题
- 审查中后台界面的设计、实现或可访问性

本 Skill 提供跨 project 的最低基线。project 中的设计系统、design token、组件封装、业务规则和术语仍是具体实现的单一事实来源。

## 参数

- 无参数：执行普通 UI/UX 开发流程，不输出正式交付检查报告。
- `delivery-check`：完成当前请求后，检查其中涉及的全部 UI/UX 变更；纯审查请求直接执行检查。
- `delivery-check=<path|page|component>`：检查参数指定的文件、页面或组件范围。

## 章节索引

Markdown 链接不会自动加载内容。每次调用先完整读取以下基础章节：

- [01 使用方式与决策优先级](references/01-usage-and-priorities.md)
- [02 设计价值观](references/02-design-values.md)

再读取任务涉及的全部章节：

- 视觉样式、主题与层级：[03 视觉基础](references/03-visual-foundations.md)
- 加载、结果、异常与系统响应：[04 反馈与状态](references/04-feedback-and-states.md)
- 菜单、Tabs、面包屑与页面返回：[05 导航](references/05-navigation.md)
- 表单、控件、校验与提交：[06 数据录入](references/06-data-entry.md)
- 表格、数据容器与图表：[07 数据展示](references/07-data-display.md)
- 界面文案、数值、时间与敏感数据：[08 文案与数据格式](references/08-content-and-data-formatting.md)
- 主次操作、按钮与风险确认：[09 按钮与操作](references/09-buttons-and-actions.md)
- 布局关系与交互行为判断：[10 交互原则](references/10-interaction-principles.md)
- 列表、详情、表单、结果、异常、工作台与看板：[11 页面模式](references/11-page-patterns.md)
- 状态变化与过渡：[12 动效](references/12-motion.md)
- 空状态、引导与品牌插图：[13 插图](references/13-illustration.md)
- 键盘、焦点、语义与响应式：[14 可访问性与适配](references/14-accessibility-and-adaptation.md)

存在 `delivery-check` 参数时，完成上述读取后再读取 [交付检查](references/delivery-check.md)。

## 开发流程

### 1. 继承 project

读取 project 中的设计系统、design token、主题配置、组件封装、同类页面、术语、权限模型与响应式约定。复用已有模式，让每项视觉和交互决策都能映射到 project 既有来源；缺失规则记录为规范缺口。

完成标准：已定位所有受影响的 project 规则、组件和同类实现。

### 2. 定义任务

明确用户角色、核心目标、使用频率、数据规模、操作风险、权限边界、主要设备和成功结果。先确定主路径，再补失败、取消、返回和恢复路径。

完成标准：能说明谁在什么场景完成什么任务、成功是什么、失败后如何继续。

### 3. 加载引用

按“章节索引”读取两份基础章节和任务涉及的全部章节；存在 `delivery-check` 参数时再读取交付检查。具体视觉取值以 project 为准，引用章节负责跨 project 的设计决策与底线。

完成标准：任务涉及的每类 UI/UX 决策都有可追溯规则。

### 4. 实现最小完整方案

使用 project 现有组件和 token 完成主任务。每个操作上下文只突出一个主操作；次要操作降级，低频操作按需展开。

完成标准：主任务可直接完成，没有重复组件、重复入口或竞争性主操作。

### 5. 验证改动

验证本次改动涉及的正常、加载、空、错误、无权限、禁用、提交中和成功状态；再验证键盘、焦点、长文本、极端数据、不同权限与窄窗口。

普通开发只报告实际执行的验证。存在 `delivery-check` 参数时，改为执行完整交付检查。

完成标准：所有受影响状态都有明确含义、反馈和恢复路径，适用验证均已通过。

## 输出

普通开发直接交付实现、关键决策和验证结果，不附带正式交付检查清单。

存在 `delivery-check` 参数时，在正常交付结果后追加 [交付检查](references/delivery-check.md) 规定的报告；纯审查请求只输出该报告。
