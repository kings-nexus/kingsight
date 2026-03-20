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

### DF-002: Mode 隔离从 prompt 层迁移到代码层
- **来源**: DP-6 升级路径
- **原始提案摘要**: 当前 agent 的单模式执行靠 skill 模板中的文字规范保证，无代码级强制。将 mode detection 迁移到 hook 或 wrapper，硬性禁止单次调用中切换模式。
- **否决理由**: 当前规模下（3 个 agent）文字规范足够，没有观察到混合模式导致质量问题
- **有价值遗产**: DP-79（质量控制最小可靠单元是代码断言）+ DP-62（连续 5 轮指令式改进无效时迁移到代码层）
- **触发条件**: 连续 >= 3 次 agent 在一次调用中混合多个模式导致质量问题
- **检查责任**: 人类手动（review agent 产出时注意）
- **检查方法**: 检查 agent 输出是否包含非当前模式的操作（如 scout 模式中出现 experiment 执行）
- **状态**: 🔍 监控中

### DF-003: Architect Phase 1.5 外部审读（跨模型独立审读产出）
- **来源**: architect_protocol_reference.md Phase 1.5，architect 设计 step 2 裁剪
- **原始提案摘要**: 让第二模型在不知道 Phase 1 发现的前提下独立审读产出，通过 Delta 对比发现主模型认知盲区
- **否决理由**: 当前文档量小（skill 模板 + 治理文档），一次 context 能读完，盲区风险低
- **有价值遗产**: Delta 对比机制（重叠=确认，Delta=盲区自动升级优先级）
- **触发条件**: 总文档（所有 SKILL.md.tmpl + docs/governance/*.md）超过 50K token
- **检查责任**: 人类手动
- **检查方法**: `wc -c */SKILL.md.tmpl docs/governance/*.md | tail -1`
- **状态**: 🔍 监控中

### DF-004: Architect Phase 2.5 Seed 对抗审查（消除 sycophancy）
- **来源**: architect_protocol_reference.md Phase 2.5，architect 设计 step 2 裁剪
- **原始提案摘要**: 用第二模型独立识别并挑战用户 seed 中的核心假设，消除主模型讨好倾向
- **否决理由**: 当前 Mode 2 输入多为假设/问题，非结构化对比表，sycophancy 风险低
- **有价值遗产**: 假设提取 + VALID/WEAK/INVALID 判定机制
- **触发条件**: architect 收到 3+ 次包含结构化对比表或多方案对比的 Mode 2 输入
- **检查责任**: 人类手动
- **检查方法**: 检查 architect-audit-log.md 中 Mode 2 条目的输入特征
- **状态**: 🔍 监控中

### DF-005: Architect Phase 3b 多轮对抗（当前 1 轮→恢复 3 轮）
- **来源**: architect_protocol_reference.md Phase 3b MIN/MAX_ROUNDS，architect 设计 step 2 裁剪
- **原始提案摘要**: 允许最多 3 轮主模型-攻击模型来回驳论，含反驳模板和再评估机制
- **否决理由**: 当前规模下 1 轮足够，项目早期提案复杂度低
- **有价值遗产**: 驳回记录格式 + Round 2+ 反驳模板 + 收敛条件定义
- **触发条件**: 击毙率连续 5 次 <10%（表明攻击不够深入，需要多轮逼出深层问题）
- **检查责任**: 人类手动
- **检查方法**: 检查最近 5 次 architect-audit-log.md 中 Phase 3 kill rate
- **状态**: 🔍 监控中

### DF-006: Architect 心跳监控协议
- **来源**: architect_protocol_reference.md 第十一节，architect 设计 step 2 裁剪
- **原始提案摘要**: 每完成关键步骤写入心跳 JSON（agent/timestamp/step/progress_pct），6 分钟无更新=异常
- **否决理由**: subagent 机制已提供基本可见性（调用方知道 subagent 何时返回）
- **有价值遗产**: 10 个关键节点的进度百分比定义
- **触发条件**: architect 运行超 15 分钟且用户无进度可见性
- **检查责任**: 人类手动
- **检查方法**: 观察 architect subagent 执行时间
- **状态**: 🔍 监控中

### DF-007: Architect 宪法审查周期
- **来源**: architect_protocol_reference.md 第七节，architect 设计 step 2 裁剪
- **原始提案摘要**: 每 10 轮运行触发全面宪法审查（7 条不变量逐条评估正面/负面证据+去除后果）
- **否决理由**: 不到 10 次 architect 运行，审查无历史数据支撑
- **有价值遗产**: 审查流程（REAFFIRM/AMEND/SUNSET）+ 修宪门槛递增机制
- **触发条件**: 累计 10 次 architect 运行
- **检查责任**: 人类手动
- **检查方法**: `grep -c "^###.*Audit" ~/.gstack/projects/$SLUG/architect-audit-log.md`
- **状态**: 🔍 监控中

### DF-008: Architect 负空间审计 + 维度覆盖度检查
- **来源**: architect_protocol_reference.md Phase 1 步骤 2.5 和 5.5，architect 设计 step 2 裁剪
- **原始提案摘要**: 5 个负空间探测问题（NQ1-NQ5）强制发现盲区 + 评估维度冗余/缺失检测
- **否决理由**: 早期阶段没有足够审计历史，负空间审计需要对"正常"有基线认知
- **有价值遗产**: NQ1-NQ5 问题模板 + "全部无盲区时强制反思审计标准是否不够严格"
- **触发条件**: 5+ 次 Mode 1 审计且 finding 数量连续 3 次下降
- **检查责任**: 人类手动
- **检查方法**: 检查 architect-audit-log.md 中 Phase 1 findings 数量趋势
- **状态**: 🔍 监控中

### DF-009: Architect 盲区问题生成 + 效能审计
- **来源**: architect_protocol_reference.md Phase 4 项 6.5 和 8，architect 设计 step 2 裁剪
- **原始提案摘要**: 效能审计（已实施提案的 eval delta 检查）+ 盲区问题生成（BSQ 格式，结构性不可自答的问题交给人类）
- **否决理由**: 无已实施提案的历史数据，盲区问题生成需要 Phase 3 积累足够分歧
- **有价值遗产**: BSQ 格式（系统决策+系统局限+人类优势+回答指引）+ Effect Rating 标准（delta>=+0.3 AND confidence=HIGH）
- **触发条件**: 3+ 个已实施提案 或 Phase 3 出现不可调和分歧
- **检查责任**: 人类手动
- **检查方法**: 检查 architect-proposals.md 中 status 为 IMPLEMENTED 的条目数
- **状态**: 🔍 监控中
