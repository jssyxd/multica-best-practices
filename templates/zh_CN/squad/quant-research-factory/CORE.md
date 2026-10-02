# quant-research-factory · 核心压缩版

## 一句话

永远在线的 Multica 量化工厂：GitHub 提策略假设 Issue → **Watchdog** 每小时领取 → **Leader** 按七步编排 → 每步独立产出 Agent + 独立审核员 → 巡查员救援 → 书记员记卡点 → 模拟盘 →（人类门禁后）实盘。

## 入口

1. 在本仓库开 **GitHub Issue**（策略假设模板见 `issue.md` / `experiments/`）
2. **WatchdogAgent-quant** 每 ~1h 扫描 open issues，未处理则 CLAIM 并在 Multica 工作区 `quantResearch` 建任务、@Leader
3. 流水线严格按 mermaid：①数据→②假设→③因子→④模型→⑤双引擎回测→⑥执行→⑦熔断 → Deploy/Monitor

## 角色（与图 1:1）

| 环 | 产出 | 审核员 |
|----|------|--------|
| 0 | Watchdog（扫仓领取） | — |
| 1 | DataAgent | ReviewerData |
| 2 | HypothesisAgent | ReviewerHypothesis |
| 3 | FactorResearcher | ReviewerFactor |
| 4 | ModelAgent | ReviewerModel |
| 5 | BacktestAgent (RD-Agent+Nautilus) | ReviewerBacktest |
| 6 | ExecutionAgent | ReviewerExecution |
| 7 | ReviewDecommission | ReviewerDecommission |
| 部署 | DeployAgent + MonitorAgent | PatrolC 协助 |

**横切：** PatrolA(1-2-3) / PatrolB(4-5-6) / PatrolC(7+部署)；Scribe；Leader 只编排。

## 硬约束（视频七步）

- 无前视；假设先于数据挖掘；因子中性/暴露说明；简单规则优先
- 真实成本 + 严格 OOS；双引擎可归因；上线当天写死 KILL
- 审核 FAIL / 幻觉 → 立即打回；卡住 >3min → 对应 Patrol

## 技术栈

Multica 编排 · RD-Agent · Qlib · XGBoost（可选）· NautilusTrader · Credal 过滤 · VibeShell · OKX/Alpaca 模拟

## Multica 工作区

- 名：`quantResearch`
- Squad：`quant-research-factory`
- 示例任务：`QUAN-1` ALPHAX-VOID-FVG-001
