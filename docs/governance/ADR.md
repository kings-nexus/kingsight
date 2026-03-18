# Architecture Decision Records

> 重大架构决策的正式记录。防止同一个决策被反复重新讨论。

## 格式

```markdown
## ADR-N: [标题]
- **日期**: YYYY-MM-DD
- **状态**: 已决定 / 已废弃
- **上下文**: [为什么需要做这个决策]
- **决定**: [我们决定做什么]
- **后果**: [这个决定带来什么影响]
- **关联**: [相关 DP/DF/文件]
```

---

## ADR-001: 独立项目，不 fork gstack 或 ECC

- **日期**: 2026-03-18
- **状态**: 已决定
- **上下文**: 项目需要 gstack 的 agent team 架构和 ECC 的自进化思想，但有完整的产品路线图（教育系统 + prompt 研究 + prompt 写作 + continuous learning），方向与两个上游项目完全不同。Fork 会带来大量不相关代码的维护负担。
- **决定**: 创建独立项目。从 gstack 和 ECC 提取设计思想和模式，不创建运行时依赖。
- **后果**: 需要自建部分基础设施（模板系统、测试框架），但获得完全的架构自主权。成熟的功能可以反向贡献给上游。
- **关联**: README-prompt-system.md

## ADR-002: Agent team 强制用 subagent，不用 CLI subprocess

- **日期**: 2026-03-18
- **状态**: 已决定
- **上下文**: 项目中存在三种 AI 调用路径（subagent / CLI subprocess / 混合）。CLI subprocess 需要独立认证（API key 或 CLI 登录），而 subagent 继承当前会话认证，零配置。E2E 测试因依赖 `claude -p` 而要求 API key，但用户从来不需要 API key。
- **决定**: Agent team 开发和测试全部使用 subagent（Agent tool）。CLI subprocess 仅用于需要多模型或有状态会话的内容生产场景。项目代码永远不引用 API key。
- **后果**: 测试零配置，开发体验一致。但无法在 subagent 中调用非 Claude 模型（需要时走 CLI subprocess 的路径 2）。
- **关联**: subproject/ai_invocation_patterns_reference.md, DP-3

## ADR-003: 一 agent 一任务

- **日期**: 2026-03-18
- **状态**: 已决定
- **上下文**: 设计 agent 角色时面临选择——让一个 agent 同时研究和写 prompt，还是拆分为两个角色。参考 DP-65（扩展现有角色 > 创建新角色）和实际需求差异（研究 = 发现规律 vs 写作 = 应用规律），两者的认知模式完全不同。
- **决定**: 每个 agent 角色做且只做一件事。Prompt researcher 只研究，prompt writer 只写，stack 只展示状态。
- **后果**: 角色职责清晰，认知模式专注。但角色数量会比"大而全"的设计更多。当两个角色的领域知识高度重叠时需要重新评估（参考 DP-65）。
- **关联**: DP-65

## ADR-004: 秘书 Agent ABCDL 五类消息分类协议

- **日期**: 2026-03-18
- **状态**: 已决定
- **上下文**: 秘书作为用户入口需要分类每条消息并路由给正确的 agent。分类基于两个正交轴：（1）谁来做——秘书自己/恢复流水线/新派发，（2）新派发的性质——探索/执行/学习。原参考设计 ABCDE 中的 E（个人决策）不适用于 prompt 工程工具，用 L（学习）替代，因为教育系统是本项目核心入口。
- **决定**: 采用 ABCDL 五类分类：A=探索未决定（→研究 agent），B=明确执行（→研究/写作 agent），C=信息查询（→秘书直答），D=流程控制（→恢复流水线），L=学习请求（→教育系统）。模糊默认 A（偏向多想而非多做）。秘书只路由不执行，只读不写。
- **后果**: 每条用户消息有明确的分类路径。A/B 需要预览门控和质量清单，C/D 轻量处理，L 是 TDL 门控的反向入口（学习解锁能力）。分类基于用户意图性质而非目标 agent，目标 agent 是分类的结果。
- **关联**: ADR-003, agent_secretary_reference.md, DP-72/73/74
