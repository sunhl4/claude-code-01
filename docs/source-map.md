# Claude Code 源码定位图

这份索引的目标不是复述报告，而是帮助你按主题快速跳到最值得读的仓库与文件。

## 1. 最优先阅读入口

| 主题 | 首选文件 | 作用 |
|---|---|---|
| Claude Code 官方公开边界 | [anthropics/claude-code/README.md](https://github.com/anthropics/claude-code/blob/main/README.md) | 先确认官方公开了什么 |
| 插件与扩展模型 | [anthropics/claude-code/plugins/README.md](https://github.com/anthropics/claude-code/blob/main/plugins/README.md) | 了解 commands/agents/skills/hooks/MCP |
| 主 runtime 入口 | [sanbuphy/claude-code-source-code/src/entrypoints/cli.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/entrypoints/cli.tsx) | CLI 入口与运行时启动点 |
| 主 agent loop | [sanbuphy/claude-code-source-code/src/query.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/query.ts) | Claude Code 最核心的执行循环 |
| QueryEngine | [sanbuphy/claude-code-source-code/src/QueryEngine.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/QueryEngine.ts) | 查询生命周期管理 |
| 系统架构总览 spec | [Kuberwastaken/claude-code/spec/00_overview.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/00_overview.md) | 快速建立结构认知 |

## 2. 按模块定位

### 2.1 入口与主循环

| 关注点 | 文件 |
|---|---|
| CLI 入口 | [src/entrypoints/cli.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/entrypoints/cli.tsx) |
| 主程序入口 | [src/main.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/main.tsx) |
| QueryEngine | [src/QueryEngine.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/QueryEngine.ts) |
| 主循环 | [src/query.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/query.ts) |
| 查询上下文 | [src/utils/queryContext.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/queryContext.ts) |
| 辅助逻辑 | [src/utils/queryHelpers.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/queryHelpers.ts) |
| 对应 spec | [spec/01_core_entry_query.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/01_core_entry_query.md) |

### 2.2 工具系统

| 关注点 | 文件 |
|---|---|
| Tool 抽象 | [src/Tool.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/Tool.ts) |
| 工具注册表 | [src/tools.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/tools.ts) |
| 工具目录 | [src/tools/](https://github.com/sanbuphy/claude-code-source-code/tree/main/src/tools) |
| 工具编排 | [src/services/tools/toolOrchestration.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/services/tools/toolOrchestration.ts) |
| 工具执行 | [src/services/tools/toolExecution.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/services/tools/toolExecution.ts) |
| Streaming tool executor | [src/services/tools/StreamingToolExecutor.js](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/services/tools/StreamingToolExecutor.js) |
| 对应 spec | [spec/03_tools.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/03_tools.md) |

### 2.3 权限系统

| 关注点 | 文件 |
|---|---|
| 权限类型 | [src/types/permissions.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/types/permissions.ts) |
| 权限逻辑 | [src/utils/permissions/permissions.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/permissions/permissions.ts) |
| 交互式权限 hook | [src/hooks/useCanUseTool.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/hooks/useCanUseTool.tsx) |
| handler 目录 | [src/hooks/toolPermission/handlers/](https://github.com/sanbuphy/claude-code-source-code/tree/main/src/hooks/toolPermission/handlers) |
| command 工具白名单 | [plugins/plugin-dev/skills/command-development/references/frontmatter-reference.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/command-development/references/frontmatter-reference.md) |
| hooks 权限扩展 | [plugins/plugin-dev/skills/hook-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/hook-development/SKILL.md) |
| 严格设置示例 | [examples/settings/settings-strict.json](https://github.com/anthropics/claude-code/blob/main/examples/settings/settings-strict.json) |

### 2.4 Commands / Agents / Skills / Hooks

| 关注点 | 文件 |
|---|---|
| 插件总览 | [plugins/README.md](https://github.com/anthropics/claude-code/blob/main/plugins/README.md) |
| plugin 结构 | [plugins/plugin-dev/skills/plugin-structure/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/plugin-structure/SKILL.md) |
| manifest 规则 | [plugins/plugin-dev/skills/plugin-structure/references/manifest-reference.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/plugin-structure/references/manifest-reference.md) |
| command frontmatter | [plugins/plugin-dev/skills/command-development/references/frontmatter-reference.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/command-development/references/frontmatter-reference.md) |
| skill 方法论 | [plugins/plugin-dev/skills/skill-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/skill-development/SKILL.md) |
| hook 方法论 | [plugins/plugin-dev/skills/hook-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/hook-development/SKILL.md) |
| plugin settings 模式 | [plugins/plugin-dev/skills/plugin-settings/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/plugin-settings/SKILL.md) |
| MCP 集成 | [plugins/plugin-dev/skills/mcp-integration/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/mcp-integration/SKILL.md) |

### 2.5 多 agent 与工作流

| 关注点 | 文件 |
|---|---|
| PR review 工作流 | [plugins/pr-review-toolkit/commands/review-pr.md](https://github.com/anthropics/claude-code/blob/main/plugins/pr-review-toolkit/commands/review-pr.md) |
| reviewer agent | [plugins/pr-review-toolkit/agents/code-reviewer.md](https://github.com/anthropics/claude-code/blob/main/plugins/pr-review-toolkit/agents/code-reviewer.md) |
| simplifier agent | [plugins/pr-review-toolkit/agents/code-simplifier.md](https://github.com/anthropics/claude-code/blob/main/plugins/pr-review-toolkit/agents/code-simplifier.md) |
| AgentTool 子 agent | [src/tools/AgentTool/runAgent.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/tools/AgentTool/runAgent.ts) |
| swarm in-process | [src/utils/swarm/inProcessRunner.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/swarm/inProcessRunner.ts) |
| spawn in-process | [src/utils/swarm/spawnInProcess.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/swarm/spawnInProcess.ts) |

### 2.6 状态、会话与 transcript

| 关注点 | 文件 |
|---|---|
| bootstrap state | [src/bootstrap/state.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/bootstrap/state.ts) |
| context | [src/context.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/context.ts) |
| session storage | [src/utils/sessionStorage.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/sessionStorage.ts) |
| AppState | [src/state/AppState.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/state/AppState.tsx) |
| AppStateStore | [src/state/AppStateStore.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/state/AppStateStore.ts) |
| services/context/state spec | [spec/06_services_context_state.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/06_services_context_state.md) |

### 2.7 hooks 与回环控制

| 关注点 | 文件 |
|---|---|
| hook 事件与输入输出 | [plugins/plugin-dev/skills/hook-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/hook-development/SKILL.md) |
| security hooks 配置 | [plugins/security-guidance/hooks/hooks.json](https://github.com/anthropics/claude-code/blob/main/plugins/security-guidance/hooks/hooks.json) |
| security hook 脚本 | [plugins/security-guidance/hooks/security_reminder_hook.py](https://github.com/anthropics/claude-code/blob/main/plugins/security-guidance/hooks/security_reminder_hook.py) |
| Ralph loop 命令 | [plugins/ralph-wiggum/commands/ralph-loop.md](https://github.com/anthropics/claude-code/blob/main/plugins/ralph-wiggum/commands/ralph-loop.md) |
| Ralph stop hook | [plugins/ralph-wiggum/hooks/stop-hook.sh](https://github.com/anthropics/claude-code/blob/main/plugins/ralph-wiggum/hooks/stop-hook.sh) |

### 2.8 SDK / MCP / bridge

| 关注点 | 文件 |
|---|---|
| SDK types | [src/entrypoints/agentSdkTypes.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/entrypoints/agentSdkTypes.ts) |
| SDK schemas | [src/entrypoints/sdk/coreSchemas.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/entrypoints/sdk/coreSchemas.ts) |
| MCP 服务层 | [src/services/mcp/](https://github.com/sanbuphy/claude-code-source-code/tree/main/src/services/mcp) |
| bridge 目录 | [src/bridge/](https://github.com/sanbuphy/claude-code-source-code/tree/main/src/bridge) |
| bridge/remote spec | [spec/09_bridge_cli_remote.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/09_bridge_cli_remote.md) |

### 2.9 UI / TUI / 产品层

| 关注点 | 文件 |
|---|---|
| components 根目录 | [src/components/](https://github.com/sanbuphy/claude-code-source-code/tree/main/src/components) |
| components spec | [spec/04_components_core_messages.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/04_components_core_messages.md) |
| agents/permissions/design spec | [spec/05_components_agents_permissions_design.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/05_components_agents_permissions_design.md) |
| Ink terminal spec | [spec/08_ink_terminal.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/08_ink_terminal.md) |

## 3. 如果只想看最有价值的 15 个文件

1. [sanbuphy/README.md](https://github.com/sanbuphy/claude-code-source-code/blob/main/README.md)
2. [sanbuphy/src/entrypoints/cli.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/entrypoints/cli.tsx)
3. [sanbuphy/src/main.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/main.tsx)
4. [sanbuphy/src/QueryEngine.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/QueryEngine.ts)
5. [sanbuphy/src/query.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/query.ts)
6. [sanbuphy/src/Tool.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/Tool.ts)
7. [sanbuphy/src/services/tools/toolOrchestration.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/services/tools/toolOrchestration.ts)
8. [sanbuphy/src/utils/permissions/permissions.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/permissions/permissions.ts)
9. [sanbuphy/src/utils/sessionStorage.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/sessionStorage.ts)
10. [anthropics/plugins/README.md](https://github.com/anthropics/claude-code/blob/main/plugins/README.md)
11. [anthropics/plugin-structure/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/plugin-structure/SKILL.md)
12. [anthropics/skill-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/skill-development/SKILL.md)
13. [anthropics/hook-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/hook-development/SKILL.md)
14. [Kuber/spec/00_overview.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/00_overview.md)
15. [Kuber/spec/INDEX.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/INDEX.md)

## 4. clean-room 复现参考

如果你的目标是自己重写一个类似 harness，可从这些文件开始：

- [instructkr/claw-code/src/main.py](https://github.com/instructkr/claw-code/blob/main/src/main.py)
- [instructkr/claw-code/src/query_engine.py](https://github.com/instructkr/claw-code/blob/main/src/query_engine.py)
- [instructkr/claw-code/src/commands.py](https://github.com/instructkr/claw-code/blob/main/src/commands.py)
- [instructkr/claw-code/src/tools.py](https://github.com/instructkr/claw-code/blob/main/src/tools.py)
- [instructkr/claw-code/rust/crates/runtime/src/lib.rs](https://github.com/instructkr/claw-code/blob/main/rust/crates/runtime/src/lib.rs)
