# 角色交付契约与 Skills（严格约束）

每个角色必须遵守：**只做本职 → 产出固定交付物 → 下一角色只消费已审核通过的交付物**。
未通过对应 Reviewer checklist 的交付物视为不存在，下游禁止读取。

通用 Skills（所有人可引用名称，由 Leader 挂载 verification）：
- `multica-verification`：Leader 通用门禁复跑
- `quant-credal-filter`：（可选）不确定性/幻觉过滤
- `quant-scribe-log`：书记员写入卡点

---

## ① DataAgent

**工作**
1. 确认 DATA_ROOT（multi-asset-ohlcv cloud_bundle）可读
2. 选定 symbol + timeframe + 日期范围；加载 Parquet 并校验 schema（open/high/low/close/volume + DatetimeIndex）
3. 清洗：重复时间戳、负价格、明显坏点；**不做**前视填充
4. 输出 point-in-time 可用的数据集清单（文件列表 + 起止时间 + bar 数）

**交付给下游（必须文件/链接）**
- `handoffs/01_data/manifest.json`：symbol, tf, files[], start, end, bar_count, tz_policy=UTC
- `handoffs/01_data/qc_report.md`：清洗规则与剔除比例
- 明确声明：**未使用未来 bar**

**禁止**
- 计算 FVG/因子/信号
- 修改假设文案

**Skills**
- `quant-data-load-parquet`
- `quant-data-qc-no-lookahead`

**下游入口条件**
- `@ReviewerData` checklist 全部 ✓ 后，`@HypothesisAgent` / `@FactorResearcher` 才可使用 manifest

---

## ② HypothesisAgent

**工作**
1. 仅基于 Issue 中 H-1 与 Pine 逻辑说明，**不读取**行情曲线优化假设
2. 写清可证伪条件与失效场景
3. 可选调用 Credal 风格自检（高不确定性则标 RISK）

**交付**
- `handoffs/02_hypothesis/H1.md`：自然语言假设 + 证伪条件
- 不得包含「根据回测年化 XX%」类事后叙事

**Skills**
- `quant-hypothesis-write`
- `quant-credal-filter`（可选）

**下游入口条件**
- `@ReviewerHypothesis` 全部 ✓ 后，`@FactorResearcher` 才可开始把 H-1 翻译成因子

---

## ③ FactorResearcher

**工作**
1. 将 FVG 检测规则形式化为可复现定义（三 K 线缺口、minGapAtr、dispMult、CE 中线、filled 判定、maxFvgAge）
2. 将 5 层汇合形式化为布尔/打分因子：EMA ribbon、Squeeze momentum、Volume delta 近似、VWAP、ADX
3. 加密标的上说明「无行业市值中性」时的替代：波动分位/BTC 贝塔暴露报告（不可假装已做股票式中性）

**交付**
- `handoffs/03_factor/factor_spec.md`：每个因子输入列、公式、窗口、无前视证明
- `handoffs/03_factor/fvg_pseudocode.py` 或等价伪代码（与 Pine 语义对齐表）
- `exposure_note.md`：暴露与局限

**Skills**
- `quant-qlib-expr`（若走 Qlib）
- `quant-fvg-spec-from-pine`
- `quant-factor-no-lookahead-check`

**下游入口条件**
- `@ReviewerFactor` ✓ → `@ModelAgent`

---

## ④ ModelAgent

**工作**
1. **默认**实现简单规则：retest + reject + score>=minConf + 过滤（session/ADX/cooldown/trend）→ 多空信号
2. 仅当 Leader 批准才引入 XGBoost；必须限制特征集为已过审因子，并做可解释性说明
3. 输出入场、SL/TP/trail 规则与 Pine 参数对照表

**交付**
- `handoffs/04_model/signal_rules.md`
- `handoffs/04_model/params_baseline.json`（禁止偷偷改 OOS 最优参）
- 若用 ML：`model_card.md`（特征、训练窗、禁止泄漏声明）

**Skills**
- `quant-signal-rules-simple`
- `quant-xgb-qlib`（可选，需批准）

**下游入口条件**
- `@ReviewerModel` ✓ → `@BacktestAgent`

---

## ⑤ BacktestAgent

**工作**
1. 同一 `manifest` + 同一 `signal_rules` 在 **引擎 A（研究：pandas/vectorbt 或 RD-Agent/Qlib）** 与 **引擎 B（NautilusTrader 事件驱动）** 跑通
2. 强制：真实费率+滑点假设写入 COST-；OOS 段不调参
3. 输出双引擎对比：信号时间戳、净值、换手、最大回撤差异归因

**交付**
- `handoffs/05_backtest/engine_a_report.md`
- `handoffs/05_backtest/engine_b_report.md`
- `handoffs/05_backtest/diff_attribution.md`
- `handoffs/05_backtest/oos_metrics.json`

**Skills**
- `quant-backtest-vector`
- `quant-backtest-nautilus`
- `quant-dual-engine-compare`

**下游入口条件**
- `@ReviewerBacktest` ✓ → `@ExecutionAgent`

---

## ⑥ ExecutionAgent

**工作**
1. 在 Nautilus（或等价）中显式建模：延迟、滑点、部分成交、涨跌停/流动性不足（加密为盘口深度简化假设）
2. 容量扫描：名义本金放大后收益衰减曲线

**交付**
- `handoffs/06_execution/impact_capacity.md`
- `CAP-` 建议最大名义本金

**Skills**
- `quant-execution-nautilus`
- `quant-capacity-scan`

**下游入口条件**
- `@ReviewerExecution` ✓ → `@ReviewDecommissionAgent`

---

## ⑦ ReviewDecommissionAgent

**工作**
1. 过程审计清单（信号是否按规则、是否冷却、是否 OOS 外偷跑参数）
2. **写死 KILL-** 到配置文件，供 Deploy/Monitor 读取

**交付**
- `handoffs/07_kill/KILL_rules.yaml`（机器可读）
- `handoffs/07_kill/audit_checklist.md`

**Skills**
- `quant-kill-switch-spec`

**下游入口条件**
- `@ReviewerDecommission` ✓ → `@DeployAgent`

---

## DeployAgent / MonitorAgent

**DeployAgent**
- Skills: `vibeshell-deploy`, `quant-sim-account-map`
- 交付：`deploy_status.json`（账户映射、进程、KILL 已加载）
- 禁止：审核未通过仍部署

**MonitorAgent**
- Skills: `quant-monitor-sim`, `quant-open-bug-issue`
- 交付：日频监控摘要；触发 KILL 时写 `triggered_kill.json`

---

## 审核员（Reviewer*）

只输出：`PASS` 或 `FAIL` + 未勾选项 + 修改方向。  
**禁止**直接改 handoffs 文件内容。  
Skills：`quant-review-checklist-<step>`

---

## 巡查员 PatrolA/B/C

- 卡住 >3 分钟或 FAIL 后停滞：主动评论并附官方文档/案例链接摘要
- Skills：`quant-patrol-docs-rag`（检索 RD-Agent、Qlib、Nautilus、VibeShell、本仓库 HANDOFF）
- 禁止：代替作者改交付物、代替审核员 PASS

---

## Scribe

- 每次门禁结果 append 到 `reports/scribe_log.md`
- Skills：`quant-scribe-log`
- 禁止：参与 PASS/FAIL 决策
