# claude-code-01

`claude-code-01` 是一个面向研究和 clean-room 学习的 Claude Code 技术资料仓库。

目标有两个：

1. 尽可能系统地整理 Claude Code 相关公开/第三方材料，形成一份可检索、可回溯的学习地图。
2. 提炼其中对通用 agent 尤其是非代码型 agent 有价值的架构思想、工程方法和模块边界。

本仓库**不再分发第三方恢复出的 Claude Code 源码**。这里保留的是：

- 研究报告
- 源码定位索引
- 对不同来源可信度和用途的分层说明
- 面向后续 agent 开发的借鉴总结

## 资料分层

### A. 官方公开仓库

- 仓库: [anthropics/claude-code](https://github.com/anthropics/claude-code)
- 作用: 官方公开的插件、commands、agents、skills、hooks、MCP 样例与 changelog
- 适合研究: 扩展机制、工作流编排方式、技能系统、hooks 设计、插件结构
- 注意: 不是完整 runtime 主源码

### B. 第三方源码快照仓库

- 仓库: [sanbuphy/claude-code-source-code](https://github.com/sanbuphy/claude-code-source-code/tree/main)
- 作用: 自称基于 npm 包 `@anthropic-ai/claude-code@2.1.88` 提取/还原的源码快照
- 适合研究: 主循环、工具系统、权限系统、任务系统、状态与上下文、桥接层
- 注意: 非官方；自述也承认缺失 feature-gated 模块，不能视为完整官方 monorepo

### C. 第三方架构分析 / spec / Rust clean-room 仓库

- 仓库: [Kuberwastaken/claude-code](https://github.com/Kuberwastaken/claude-code)
- 作用: 长篇分析文章 + `spec/` + Rust clean-room 结构
- 适合研究: 架构解读、主题索引、子系统 spec、对隐藏能力的分析线索
- 注意: README 中有不少叙述性内容，需与 `spec/` 和其他来源交叉验证

### D. clean-room 重写项目

- 仓库: [instructkr/claw-code](https://github.com/instructkr/claw-code)
- 作用: Python-first / Rust in progress 的 clean-room rewrite
- 适合研究: 怎样把大规模 agent harness 拆解后再实现；哪些部分被优先复现
- 注意: 当前重点已转向 rewrite，本身不是 Claude Code 原始源码镜像

## 文档索引

- 主报告: [`docs/claude-code-study-report.zh-CN.md`](docs/claude-code-study-report.zh-CN.md)
- 源码定位图: [`docs/source-map.md`](docs/source-map.md)
- 面向后续 agent 的借鉴总结: [`docs/agent-design-takeaways.md`](docs/agent-design-takeaways.md)

## 建议阅读顺序

1. 先读主报告，建立整体分层认知。
2. 再读源码定位图，按关注模块跳到对应仓库与文件。
3. 最后读借鉴总结，把 Claude Code 的方法映射到自己的 agent 系统。

## 使用方式

如果你的目标是“学习 Claude Code”，建议从以下顺序开始：

1. 官方公开扩展层
2. 第三方源码快照中的主运行时
3. spec / clean-room 重写中的结构化总结
4. 再回到自己的 agent 设计做映射

如果你的目标是“做自己的 agent”，建议重点看：

1. `query / QueryEngine / tools / permissions / tasks`
2. `commands / agents / skills / hooks / MCP`
3. `session / transcript / compact / state`
4. clean-room rewrite 如何抽离出最核心的 harness
