# Deferred Items

> 被否决但有潜在价值的提案。设触发条件，条件满足时复活。

## 语义边界

- 活跃任务放在 call stack（/stack）
- 延迟项放在这里（被否决但有价值的遗产）
- 两者不重叠

## 格式

```markdown
### DF-N: [一句话标题]
- **来源**: [被否决的提案/讨论]
- **原始提案摘要**: [≤100字]
- **否决理由**: [一句话]
- **有价值遗产**: [替代方案 / 调研需求 / 条件触发定义]
- **触发条件**: [必须是可观测事件，不是时间流逝]
- **状态**: 🔍 监控中 / ✅ 已触发(→新条目) / ❌ 已失效
```

---

## 当前延迟项

### DF-001: Continuous Learning 自动化（Hook 观察 + 背景分析）
- **来源**: ECC continuous-learning-v2 设计评估
- **原始提案摘要**: 通过 PreToolUse/PostToolUse hook 自动捕获每次交互，背景 Haiku agent 提取模式为 instinct，置信度驱动自动升级
- **否决理由**: 当前 3 个 skill、0 个用户，没有足够交互数据，手动 distill 完全够用
- **有价值遗产**: 架构设计可参考 ECC v2.1（项目隔离、置信度评分、进化管线）。当前手动流程（secretary 日志 + prompt-research distill）是未来自动化的数据基础
- **触发条件**: 用户数 >= 5 或 secretary 日志条目 >= 100 条（表明有足够数据密度值得自动分析）
- **检查责任**: 人类手动
- **检查方法**: `wc -l ~/.gstack/projects/$SLUG/secretary-journal.jsonl`
- **状态**: 🔍 监控中
