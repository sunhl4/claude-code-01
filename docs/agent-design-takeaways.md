# Claude Code 对后续 Agent 开发的借鉴

这份文档不再重复源码定位，而是直接回答一个问题：

Claude Code 最值得迁移到你自己 agent 系统里的是什么？

## 1. 不要只做 Prompt Engineering，要做 Harness Engineering

Claude Code 给人的最大启发，不是某个 prompt 写得多强，而是它把 agent 做成了“运行时 + 工具 + 权限 + 状态 + 扩展”的完整系统。

最值得看的代码入口：

- 主循环: [sanbuphy/src/query.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/query.ts)
- QueryEngine: [sanbuphy/src/QueryEngine.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/QueryEngine.ts)
- 工具编排: [sanbuphy/src/services/tools/toolOrchestration.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/services/tools/toolOrchestration.ts)
- 权限逻辑: [sanbuphy/src/utils/permissions/permissions.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/permissions/permissions.ts)

迁移建议：

- 先设计运行时，再设计 prompt。
- 把“工具调用、任务切分、上下文压缩、状态保存”视作一等能力。

## 2. 用 Commands / Agents / Skills / Hooks 四层拆能力

官方公开仓库展示得最完整的是这一点。

关键文档：

- 插件结构: [anthropics/plugin-structure/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/plugin-structure/SKILL.md)
- command 规则: [anthropics/command frontmatter](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/command-development/references/frontmatter-reference.md)
- skill 规则: [anthropics/skill-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/skill-development/SKILL.md)
- hook 规则: [anthropics/hook-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/hook-development/SKILL.md)

一个通用 agent 系统可以照着拆：

- `Commands`: 用户显式触发的任务模板
- `Agents`: 专家角色
- `Skills`: 知识和流程包
- `Hooks`: 生命周期治理

## 3. 知识要按需加载，不要一次性塞满上下文

Claude Code 的 skill 体系最值得借鉴的是 progressive disclosure。

代码定位：

- [anthropics/skill-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/skill-development/SKILL.md)

迁移建议：

- 永久常驻的只保留 skill metadata
- 实际触发时再加载 skill 主文档
- 大参考资料、模板、脚本按需再读

适用场景：

- 法律 agent
- 医疗辅助 agent
- 研究 agent
- 企业知识工作流 agent

## 4. 生命周期钩子是强 agent 的核心差异

Claude Code 的 hook 体系说明，一个真正可控的 agent 不应该只有“输入 -> 输出”。

代码定位：

- [anthropics/hook-development/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/hook-development/SKILL.md)
- [anthropics/security hook](https://github.com/anthropics/claude-code/blob/main/plugins/security-guidance/hooks/security_reminder_hook.py)
- [anthropics/ralph stop hook](https://github.com/anthropics/claude-code/blob/main/plugins/ralph-wiggum/hooks/stop-hook.sh)

建议你未来的 agent 至少保留这些 hook 点：

- `SessionStart`
- `UserPromptSubmit`
- `PreToolUse`
- `PostToolUse`
- `Stop`
- `PreCompact`

## 5. 权限一定要做双层

Claude Code 的经验说明，光靠模型“自觉”远远不够。

代码定位：

- [sanbuphy/src/types/permissions.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/types/permissions.ts)
- [sanbuphy/src/utils/permissions/permissions.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/permissions/permissions.ts)
- [anthropics/examples/settings/settings-strict.json](https://github.com/anthropics/claude-code/blob/main/examples/settings/settings-strict.json)

建议采用：

- 静态能力白名单
- 运行时审批或 deny/ask/allow
- hook 级治理
- 组织托管规则

## 6. Specialist Agents 比万能 Agent 更可控

最典型例子是 PR review toolkit。

代码定位：

- [anthropics/review-pr command](https://github.com/anthropics/claude-code/blob/main/plugins/pr-review-toolkit/commands/review-pr.md)
- [anthropics/code-reviewer](https://github.com/anthropics/claude-code/blob/main/plugins/pr-review-toolkit/agents/code-reviewer.md)
- [anthropics/code-simplifier](https://github.com/anthropics/claude-code/blob/main/plugins/pr-review-toolkit/agents/code-simplifier.md)

迁移建议：

- 主 agent 负责接收用户任务和汇总
- 子 agent 负责证据抽取、事实核验、风险识别、风格审校、输出润色等专职工作

## 7. transcript 和状态文件要显式存在

强 agent 要能恢复、审计、复盘、循环执行。

代码定位：

- [sanbuphy/src/utils/sessionStorage.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/utils/sessionStorage.ts)
- [sanbuphy/src/bootstrap/state.ts](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/bootstrap/state.ts)
- [anthropics/plugin-settings/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/plugin-settings/SKILL.md)
- [anthropics/ralph-loop](https://github.com/anthropics/claude-code/blob/main/plugins/ralph-wiggum/commands/ralph-loop.md)

建议：

- transcript 用于审计和回放
- 本地状态文件用于项目级配置和循环状态
- runtime store 用于进程内状态

## 8. MCP / Connectors 要做成总线，不是零散 API

代码定位：

- [anthropics/mcp-integration/SKILL.md](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/mcp-integration/SKILL.md)
- [sanbuphy/src/services/mcp](https://github.com/sanbuphy/claude-code-source-code/tree/main/src/services/mcp)

迁移建议：

- 给每个外部系统定义统一注册和命名空间
- 让权限系统能识别具体连接器和具体工具
- 让日志和 transcript 能看到调用来源

## 9. UI 不是附属品，而是运行时的一部分

Claude Code 很强的一点是它把 agent 状态可视化了。

代码定位：

- [sanbuphy/src/main.tsx](https://github.com/sanbuphy/claude-code-source-code/blob/main/src/main.tsx)
- [sanbuphy/src/components](https://github.com/sanbuphy/claude-code-source-code/tree/main/src/components)
- [Kuber/spec/04_components_core_messages.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/04_components_core_messages.md)
- [Kuber/spec/08_ink_terminal.md](https://github.com/Kuberwastaken/claude-code/blob/main/spec/08_ink_terminal.md)

这对非代码 agent 同样成立：

- 执行状态
- 审批节点
- 工具调用回显
- 长流程进度
- 失败与恢复信息

都应该是产品设计的一部分。

## 10. clean-room 重建路线可以怎么走

如果你未来想参考 Claude Code 做自己的系统，最稳的路线是：

1. 先读官方公开扩展层
2. 再读第三方源码快照中的 runtime 主干
3. 用 spec 帮助建立模块地图
4. 参考 clean-room rewrite 的抽象方法重建自己的系统

对应参考：

- [anthropics/claude-code](https://github.com/anthropics/claude-code)
- [sanbuphy/claude-code-source-code](https://github.com/sanbuphy/claude-code-source-code/tree/main)
- [Kuberwastaken/claude-code](https://github.com/Kuberwastaken/claude-code)
- [instructkr/claw-code](https://github.com/instructkr/claw-code)

## 11. 一句话总结

Claude Code 最值得借鉴的不是“它会写代码”，而是：

**它把 agent 做成了一个可以运行、可以治理、可以扩展、可以协作、可以恢复的软件系统。**
