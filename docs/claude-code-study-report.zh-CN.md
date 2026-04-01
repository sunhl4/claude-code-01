# Claude Code 技术研究报告

## 0. 报告目的

这份报告面向两个目标：

1. 作为 Claude Code 学习地图，帮助读者快速定位最重要的源码与文档入口。
2. 作为后续 agent 开发参考，提炼 Claude Code 在架构、工具、权限、状态、扩展、协作上的关键思想。

本报告采用“来源分层”的方法，不把不同可信度的材料混为一谈。

## 1. 资料来源与可信度分层

### 1.1 官方公开层

官方仓库主要公开的是扩展层，而不是完整 runtime 主体。

代码入口与说明：

- 官方仓库 README: [anthropics/claude-code/README.md](https://github.com/anthropics/claude-code/blob/main/README.md)
- 官方插件目录总览: [anthropics/claude-code/plugins/README.md](https://github.com/anthropics/claude-code/blob/main/plugins/README.md)
- 官方插件结构文档: [anthropics/claude-code/plugins/plugin-dev/skills/plugin-structure/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/plugin-structure/SKILL.md)

结论：

- 它足以说明 Claude Code 的公开扩展模型是什么。
- 它不足以独立还原 Claude Code 完整主运行时。

### 1.2 第三方源码快照层

`sanbuphy/claude-code-source-code` 自称基于 npm 包恢复出较完整的 TypeScript 源码快照，并且目录结构明显包含主程序运行时切面。

代码入口与说明：

- 仓库 README: [sanbuphy/claude-code-source-code/README.md](https://github.com/sanbuphy/claude-code-source-code/blob/main/README.md)
- `package.json`: [sanbuphy/claude-code-source-code/package.json](https://github.com/sanbuphy/claude-code-source-code/blob/main/package.json)
- CLI 入口: [sanbuphy/claude-code-source-code/src/entrypoints/cli.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/entrypoints/cli.tsx)
- 主运行时入口: [sanbuphy/claude-code-source-code/src/main.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/main.tsx)
- 主 agent loop: [sanbuphy/claude-code-source-code/src/query.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/query.ts)
- QueryEngine: [sanbuphy/claude-code-source-code/src/QueryEngine.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/QueryEngine.ts)

结论：

- 它是当前最接近“Claude Code 主体工程结构”的可研究材料。
- 但它不是官方仓库，也不是完整内部 monorepo。
- 研究时应把它视为“高价值非官方源码快照”。

### 1.3 第三方 spec / 架构分析层

`Kuberwastaken/claude-code` 更像“spec + clean-room Rust 实现 + 文章化分析”。

代码入口与说明：

- 仓库 README: [Kuberwastaken/claude-code/README.md](https://github.com/Kuberwastaken/claude-code/blob/main/README.md)
- Spec 索引: [Kuberwastaken/claude-code/spec/INDEX.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/INDEX.md)
- 总览 spec: [Kuberwastaken/claude-code/spec/00_overview.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/00_overview.md)
- Core/query spec: [Kuberwastaken/claude-code/spec/01_core_entry_query.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/01_core_entry_query.md)
- Tools spec: [Kuberwastaken/claude-code/spec/03_tools.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/03_tools.md)
- Services/context/state spec: [Kuberwastaken/claude-code/spec/06_services_context_state.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/06_services_context_state.md)
- Rust codebase spec: [Kuberwastaken/claude-code/spec/13_rust_codebase.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/13_rust_codebase.md)

结论：

- 它适合拿来做“结构化理解”和快速索引。
- 它不应被当作每个叙述性细节都已被独立验证的权威来源。

### 1.4 clean-room 重写层

`instructkr/claw-code` 现在更像一个 clean-room harness 重写项目，而不是原始 Claude Code 归档。

代码入口与说明：

- 仓库 README: [instructkr/claw-code/README.md](https://github.com/instructkr/claw-code/blob/main/README.md)
- Python CLI: [instructkr/claw-code/src/main.py](https://github.com/instructkr/claw-code/blob/main/src/main.py)
- Python query engine: [instructkr/claw-code/src/query_engine.py](https://github.com/instructkr/claw-code/blob/main/src/query_engine.py)
- Python commands metadata: [instructkr/claw-code/src/commands.py](https://github.com/instructkr/claw-code/blob/main/src/commands.py)
- Python tools metadata: [instructkr/claw-code/src/tools.py](https://github.com/instructkr/claw-code/blob/main/src/tools.py)
- Rust runtime crate: [instructkr/claw-code/rust/crates/runtime/src/lib.rs](https://github.com/instructkr/claw-code/blob/main/rust/crates/runtime/src/lib.rs)

结论：

- 它非常适合研究“如何 clean-room 地重建一个 agent harness”。
- 它不适合作为原始 Claude Code 源码的唯一依据。

## 2. Claude Code 的整体架构观

从第三方源码快照和 spec 来看，Claude Code 可以理解为一个“宿主运行时 + 工具系统 + 状态系统 + 扩展系统 + 桥接层”的大型 agent harness。

最关键的主干文件是：

- CLI 入口: [src/entrypoints/cli.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/entrypoints/cli.tsx)
- 主程序入口: [src/main.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/main.tsx)
- 核心会话编排: [src/QueryEngine.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/QueryEngine.ts)
- 主循环: [src/query.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/query.ts)
- 工具接口: [src/Tool.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/Tool.ts)
- 工具集合: [src/tools.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/tools.ts)

这套结构说明 Claude Code 并不是一个“单 prompt CLI”，而是一个真正的运行时系统。它做了几件事：

- 管理输入与消息流
- 组装系统提示与上下文
- 向模型发起流式请求
- 识别和调度工具调用
- 维护会话状态、权限、成本、压缩与持久化
- 支持子 agent、任务、后台工作与桥接

对应的 spec 快速导览：

- 总览: [spec/00_overview.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/00_overview.md)
- 查询主线: [spec/01_core_entry_query.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/01_core_entry_query.md)

## 3. 核心主循环：Claude Code 本质上是一个生产级 agent loop

Claude Code 最值得学习的，不是它会写代码，而是它把一个基础 agent loop 做成了工业级版本。

最关键的研究入口：

- `query`: [src/query.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/query.ts)
- `QueryEngine`: [src/QueryEngine.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/QueryEngine.ts)
- query context: [src/utils/queryContext.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/queryContext.ts)
- query helpers: [src/utils/queryHelpers.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/queryHelpers.ts)

这一层大致负责：

- 接收用户消息
- 归并会话消息
- 构建系统 prompt 与工具描述
- 发起模型调用
- 流式消费响应
- 检测 `tool_use`
- 执行工具并将结果重新塞回消息流
- 在上下文超限时触发压缩
- 最终产出文本结果或进入下一轮工具-模型循环

这说明 Claude Code 的底座，不是“命令行版聊天”，而是“循环式代理执行器”。

对后续 agent 开发的借鉴：

- 把模型调用、工具执行、状态管理、压缩和持久化都视为主循环的一部分。
- 不要把工具系统设计成旁路补丁，而要把它融入 turn loop。

## 4. 工具系统：Claude Code 的能力不是 prompt，而是可调度工具网络

工具架构的核心文件：

- 工具接口: [src/Tool.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/Tool.ts)
- 工具注册: [src/tools.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/tools.ts)
- 工具编排: [src/services/tools/toolOrchestration.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/services/tools/toolOrchestration.ts)
- 工具执行: [src/services/tools/toolExecution.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/services/tools/toolExecution.ts)
- 流式工具执行器: [src/services/tools/StreamingToolExecutor.js](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/services/tools/StreamingToolExecutor.js)

按公开资料和快照来看，Claude Code 的工具不是几个简单命令，而是一个完整的工具平台。它至少包括：

- 文件读写编辑
- shell/bash/powershell
- 搜索与 glob
- web fetch / web search
- MCP
- task / team / message
- ask user
- plan / worktree / todo

对应 spec：

- 工具总表: [spec/03_tools.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/03_tools.md)

工程思想：

- 每个工具都是标准化接口，不是自由散落的函数。
- 工具具有输入校验、执行、权限、展示、结果映射等生命周期。
- 工具可以并发或串行地被调度。

这对你的后续 agent 非常重要。你做的 agent 如果不是 code agent，也依然应该把“检索、审批、消息、数据库、文件、流程节点、外部 API”统一抽象成工具接口层。

## 5. 权限系统：真正强的 agent 一定有多层权限控制

关键入口：

- 权限类型: [src/types/permissions.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/types/permissions.ts)
- 权限判断核心: [src/utils/permissions/permissions.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/permissions/permissions.ts)
- UI/交互入口: [src/hooks/useCanUseTool.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/hooks/useCanUseTool.tsx)
- 交互式处理器: [src/hooks/toolPermission/handlers/interactiveHandler.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/hooks/toolPermission/handlers/interactiveHandler.ts)

公开仓库的扩展层也提供了一个更容易理解的视角：

- Command 工具白名单: [plugins/plugin-dev/skills/command-development/references/frontmatter-reference.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/command-development/references/frontmatter-reference.md)
- Hook 事件与决策输出: [plugins/plugin-dev/skills/hook-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/hook-development/SKILL.md)
- 严格权限示例: [examples/settings/settings-strict.json](https://github.com/anthropics/claude-code/blob/main/examples/settings/settings-strict.json)

Claude Code 的权限模型值得借鉴的地方在于：

- 静态边界: `allowed-tools`
- 动态边界: `PreToolUse` hook
- 运行时模式: `allow / ask / deny`
- 组织级或托管规则

这意味着强 agent 不是“默认全能”，而是“能力分配 + 运行时许可”。

## 6. Commands / Agents / Skills / Hooks：Claude Code 的扩展模型

如果说主循环是底座，那么公开官方仓库最好地展示了 Claude Code 的扩展层。

### 6.1 插件结构

- 总览: [plugins/README.md](https://github.com/anthropics/claude-code/blob/main/plugins/README.md)
- 插件结构: [plugins/plugin-dev/skills/plugin-structure/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/plugin-structure/SKILL.md)
- manifest 参考: [plugins/plugin-dev/skills/plugin-structure/references/manifest-reference.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/plugin-structure/references/manifest-reference.md)

结论：

- Claude Code 把可扩展能力拆成插件。
- 插件由 `plugin.json`、`commands/`、`agents/`、`skills/`、`hooks/`、`.mcp.json` 构成。

### 6.2 Commands

- frontmatter 规则: [plugins/plugin-dev/skills/command-development/references/frontmatter-reference.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/command-development/references/frontmatter-reference.md)
- 真实命令例子: [plugins/pr-review-toolkit/commands/review-pr.md](https://github.com/anthropics/claude-code/blob/main/plugins/pr-review-toolkit/commands/review-pr.md)

Claude Code 的 command 更像“声明式任务单元”而不是 shell alias。它定义：

- 做什么
- 用什么工具
- 用哪个模型
- 接收什么参数

### 6.3 Agents

- 真实 agent 例子: [plugins/pr-review-toolkit/agents/code-reviewer.md](https://github.com/anthropics/claude-code/blob/main/plugins/pr-review-toolkit/agents/code-reviewer.md)
- 另一个 agent 例子: [plugins/pr-review-toolkit/agents/code-simplifier.md](https://github.com/anthropics/claude-code/blob/main/plugins/pr-review-toolkit/agents/code-simplifier.md)
- agent 设计文档: [plugins/plugin-dev/skills/agent-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/agent-development/SKILL.md)

这说明 Claude Code 倾向于：

- 把复杂工作拆给专职 agent
- 主 agent 负责编排
- 子 agent 聚焦 narrow scope

### 6.4 Skills

- skill 方法论: [plugins/plugin-dev/skills/skill-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/skill-development/SKILL.md)

skills 最重要的思想是渐进式加载：

- metadata 常驻
- `SKILL.md` 在触发时加载
- `references/`、`scripts/`、`assets/` 按需读

这非常适合非代码 agent 的知识包设计。

### 6.5 Hooks

- hook 文档: [plugins/plugin-dev/skills/hook-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/hook-development/SKILL.md)
- security hook 示例: [plugins/security-guidance/hooks/hooks.json](https://github.com/anthropics/claude-code/blob/main/plugins/security-guidance/hooks/hooks.json)
- security hook 实现: [plugins/security-guidance/hooks/security_reminder_hook.py](https://github.com/anthropics/claude-code/blob/main/plugins/security-guidance/hooks/security_reminder_hook.py)
- ralph loop stop hook: [plugins/ralph-wiggum/hooks/stop-hook.sh](https://github.com/anthropics/claude-code/blob/main/plugins/ralph-wiggum/hooks/stop-hook.sh)

Claude Code 用 hooks 管理：

- 输入前治理
- 工具前审批
- 工具后反馈
- 停止前完整性校验
- 会话开始/结束
- compact 前信息保留

它把 agent 从“会聊天的工具”提升成“有生命周期治理的系统”。

## 7. Session、状态、transcript 与 context compaction

这部分是很多 agent 系统最薄弱、但 Claude Code 最像工程产品的地方。

关键定位：

- context 入口: [src/context.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/context.ts)
- bootstrap state: [src/bootstrap/state.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/bootstrap/state.ts)
- session storage: [src/utils/sessionStorage.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/sessionStorage.ts)
- AppState: [src/state/AppState.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/state/AppState.tsx)
- AppStateStore: [src/state/AppStateStore.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/state/AppStateStore.ts)

Spec 对应：

- services/context/state: [spec/06_services_context_state.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/06_services_context_state.md)

公开插件层还能看到一些“状态文件”模式：

- plugin settings: [plugins/plugin-dev/skills/plugin-settings/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/plugin-settings/SKILL.md)
- ralph loop 本地状态: [plugins/ralph-wiggum/commands/ralph-loop.md](https://github.com/anthropics/claude-code/blob/main/plugins/ralph-wiggum/commands/ralph-loop.md)

观察到的重要思想：

- transcript 是一等公民
- session id 明确存在
- 状态不仅存在于模型上下文，也存在于显式文件和 store 中
- compaction 是系统机制，不是临时补救

对后续 agent 借鉴：

- 把会话日志、任务状态、短期记忆、长期配置拆开
- 不要把所有记忆都塞给模型上下文

## 8. 多 agent、任务系统与协作模式

关键文件：

- AgentTool 子 agent 运行: [src/tools/AgentTool/runAgent.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/tools/AgentTool/runAgent.ts)
- swarm in-process: [src/utils/swarm/inProcessRunner.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/swarm/inProcessRunner.ts)
- spawn in-process: [src/utils/swarm/spawnInProcess.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/swarm/spawnInProcess.ts)
- 任务命令目录: [src/commands/tasks](https://github.com/sanbuphy/claude-code-source-code/tree/main/src/commands/tasks)
- task 输出相关: [src/utils/task/TaskOutput.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/task/TaskOutput.ts)

官方公开层的直观样例：

- PR review command: [plugins/pr-review-toolkit/commands/review-pr.md](https://github.com/anthropics/claude-code/blob/main/plugins/pr-review-toolkit/commands/review-pr.md)
- code reviewer agent: [plugins/pr-review-toolkit/agents/code-reviewer.md](https://github.com/anthropics/claude-code/blob/main/plugins/pr-review-toolkit/agents/code-reviewer.md)

可得出几个结论：

- Claude Code 的多 agent 不是“实验性玩具”，而是工作流一部分。
- 主 agent 负责切问题和汇总结果。
- 子 agent 负责深挖特定维度。
- 任务与消息机制支撑协作，而不是靠 prompt 暗示。

## 9. Bridge、SDK、MCP 与外部系统接入

如果一个 agent 要进入 IDE、桌面端、远程端或企业系统，就必须有桥接层与接入层。

快照仓库中的关键入口：

- SDK types: [src/entrypoints/agentSdkTypes.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/entrypoints/agentSdkTypes.ts)
- SDK core schemas: [src/entrypoints/sdk/coreSchemas.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/entrypoints/sdk/coreSchemas.ts)
- MCP 相关目录: [src/services/mcp](https://github.com/sanbuphy/claude-code-source-code/tree/main/src/services/mcp)
- bridge 相关目录: [src/bridge](https://github.com/sanbuphy/claude-code-source-code/tree/main/src/bridge)

官方公开层中的 MCP 模式：

- MCP integration skill: [plugins/plugin-dev/skills/mcp-integration/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/mcp-integration/SKILL.md)

可借鉴点：

- SDK / bridge 让 agent 能脱离单一终端环境
- MCP / connector 让能力来源标准化
- 命名空间化工具名有助于权限和治理

## 10. UI 与产品化：Claude Code 不只是 API 包装器

快照仓库和 spec 都说明 Claude Code 有完整的 TUI / React / Ink 层。

关键入口：

- 主入口: [src/main.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/main.tsx)
- components 目录: [src/components](https://github.com/sanbuphy/claude-code-source-code/tree/main/src/components)
- state provider: [src/state/AppState.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/state/AppState.tsx)
- UI/Ink spec: [spec/04_components_core_messages.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/04_components_core_messages.md)
- Ink terminal spec: [spec/08_ink_terminal.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/08_ink_terminal.md)

这说明 Claude Code 的优势不只来自模型，也来自：

- 可见的执行状态
- 工具与权限的可交互反馈
- 长任务的 UI 表达
- 会话与消息渲染体验

很多 agent 产品最后输掉，不是智能不够，而是运行时体验太弱。

## 11. Claude Code 最值得迁移到非代码 agent 的思想

### 11.1 宿主运行时和领域包分离

代码定位：

- `query / QueryEngine / Tool / tools.ts`
- `commands / agents / skills / hooks`

解释：

- 宿主运行时只负责编排和治理
- 具体领域能力用插件/skill/agent/command 提供

### 11.2 生命周期治理优先

代码定位：

- [hook-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/hook-development/SKILL.md)

解释：

- 输入、工具前、工具后、停止前、compact 前都应有钩子

### 11.3 知识按需加载

代码定位：

- [skill-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/skill-development/SKILL.md)

解释：

- 别把所有知识都塞进系统提示
- 用 metadata + core skill + references 三层加载

### 11.4 specialist agents 代替万能 agent

代码定位：

- [pr-review-toolkit/commands/review-pr.md](https://github.com/anthropics/claude-code/blob/main/plugins/pr-review-toolkit/commands/review-pr.md)
- [pr-review-toolkit/agents/code-reviewer.md](https://github.com/anthropics/claude-code/blob/main/plugins/pr-review-toolkit/agents/code-reviewer.md)

解释：

- 主 agent 做协调
- 专家 agent 做特化分析

### 11.5 transcript 和状态文件是一等公民

代码定位：

- [src/utils/sessionStorage.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/sessionStorage.ts)
- [plugins/plugin-dev/skills/plugin-settings/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/plugin-settings/SKILL.md)
- [plugins/ralph-wiggum/hooks/stop-hook.sh](https://github.com/anthropics/claude-code/blob/main/plugins/ralph-wiggum/hooks/stop-hook.sh)

解释：

- 把会话与状态显式化，后续调试、审计、恢复都更容易

## 12. 建议你的学习路径

### 路线 A：先学公开扩展层

1. [plugins/README.md](https://github.com/anthropics/claude-code/blob/main/plugins/README.md)
2. [plugin-structure/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/plugin-structure/SKILL.md)
3. [command frontmatter](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/command-development/references/frontmatter-reference.md)
4. [skill-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/skill-development/SKILL.md)
5. [hook-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/hook-development/SKILL.md)

### 路线 B：直接研究主 runtime

1. [src/entrypoints/cli.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/entrypoints/cli.tsx)
2. [src/main.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/main.tsx)
3. [src/QueryEngine.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/QueryEngine.ts)
4. [src/query.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/query.ts)
5. [src/Tool.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/Tool.ts)
6. [src/services/tools/toolOrchestration.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/services/tools/toolOrchestration.ts)
7. [src/utils/permissions/permissions.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/permissions/permissions.ts)
8. [src/utils/sessionStorage.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/sessionStorage.ts)

### 路线 C：借助 spec 提高阅读效率

1. [spec/INDEX.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/INDEX.md)
2. [spec/00_overview.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/00_overview.md)
3. [spec/01_core_entry_query.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/01_core_entry_query.md)
4. [spec/03_tools.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/03_tools.md)
5. [spec/06_services_context_state.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/06_services_context_state.md)
6. [spec/13_rust_codebase.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/13_rust_codebase.md)

## 13. 最终结论

Claude Code 最值得学习的，不是某一个“隐藏 feature”，而是它把 agent 做成了一个完整的软件系统：

- 有运行时
- 有工具层
- 有权限层
- 有状态层
- 有扩展层
- 有协作层
- 有 UI 层
- 有桥接层

对后续 agent 开发来说，Claude Code 的真正启发是：

- 不要只做 prompt 工程
- 要做 harness engineering
- 不要只做单 agent
- 要做可治理、可扩展、可恢复、可协作的 agent 系统
