# quant-research-factory

严格对齐视频《一条量化策略到底是怎样做出来的？》七步研究链路，并叠加：

- **每个子环节后独立审核员**（详细检查清单，全部 check 通过才放行，错误/幻觉立即打回）
- **3 个流程巡查员**（A=1-2-3，B=4-5-6，C=7+部署模拟盘；主动学习官方文档与优秀案例；解救卡住>3分钟或被打回的 Agent）
- **1 个独立书记员**（全程记录卡点、错误点、打回原因，供人工优化工作流）

技术栈约束：Multica 编排 + Microsoft RD-Agent 研究内核 + Qlib 因子库 + XGBoost + 双引擎回测（RD-Agent ↔ NautilusTrader）+ Credal 反幻觉 + VibeShell 部署 + 20 模拟账户（10 OKX + 10 Alpaca）。

## 什么时候用

- 你要做**全自动量化研究策略工厂**，从假设到模拟盘再到小资金实盘。
- 必须拒绝「单点调参 / 先射箭再画靶」，强制七步系统工程思维。
- 需要双引擎交叉验证、风格剥离、真实成本、严格 OOS、预设熔断。

## 七步 ↔ 角色 ↔ 审核员映射（1:1 图表）

| 步骤 | 产出 Agent | 独立审核员 | 检查清单核心 |
| --- | --- | --- | --- |
| ① 数据 | @DataAgent | @ReviewerData | 无前视偏差、按披露日回填、复权/停牌正确 |
| ② 假设 | @HypothesisAgent | @ReviewerHypothesis | 假设先于数据、经济逻辑可证伪、Credal 合格 |
| ③ 因子 | @FactorResearcher | @ReviewerFactor | 规模中性、标准化、行业+市值风格剥离 |
| ④ 模型 | @ModelAgent | @ReviewerModel | 简单规则优先、参数未爆炸、可解释 |
| ⑤ 回测 | @BacktestAgent（双引擎） | @ReviewerBacktest | 全周期、真实成本、严格 OOS、双引擎一致 |
| ⑥ 执行 | @ExecutionAgent | @ReviewerExecution | 冲击成本、流动性/涨跌停、容量衰减 |
| ⑦ 复盘下线 | @ReviewDecommissionAgent | @ReviewerDecommission | 过程审计、风格漂移监控、上线当天写死熔断 |
| 部署模拟 | @DeployAgent + @MonitorAgent | （由巡查员 C + 书记员覆盖） | VibeShell 部署、20 模拟账户、熔断触发 |

## 巡查员与书记员

| 角色 | 负责板块 | 职责 |
| --- | --- | --- |
| @PatrolA | 1-2-3 | 主动学习 Qlib/数据清洗/假设官方文档与案例；解救卡住>3min 或被打回 |
| @PatrolB | 4-5-6 | 主动学习 RD-Agent/Nautilus/回测执行文档与案例；同上 |
| @PatrolC | 7 + 部署模拟 | 主动学习 VibeShell/风控熔断/模拟盘文档与案例；同上 |
| @Scribe | 全程 | 记录卡点、错误、打回原因、巡查介入，输出优化报告 |

## 配套文件

- `squad.md`：完整 Squad 指令（严格 1:1 图表）
- `issue.md`：量化策略 Issue 模板
- 对应 Agent 指令见 `templates/zh_CN/agents/` 下 quant-* 系列（若尚未同步，先用本 README 与 squad.md 手工创建）

## 与 software-development-reviewed 的关系

复用其「两层门禁」思想（Leader 通用门禁 + 专属审核员专业清单），但角色与阶段全部替换为量化七步，并新增巡查员与书记员。
