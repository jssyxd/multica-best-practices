# 实验 Issue：ALPHAX-VOID-FVG-001

> 复制到 Multica Issue。策略来源：TradingView Pine Script「AlphaX VOID v1.0」Fair Value Gap Confluence System。

```text
【策略代号】
ALPHAX-VOID-FVG-001

【一句话目标】
将 TradingView 上的 FVG（公允价值缺口）+ 5 层汇合（EMA Ribbon / Squeeze Momentum / Volume Delta / VWAP / ADX）回测再入场逻辑，在可复现的历史 OHLCV 上完成七步研究链路，双引擎交叉验证后进入模拟盘。

【自然语言假设 H-1（先于任何深度数据挖掘）】
在出现足够大的价格位移形成的未回填 Fair Value Gap 后，价格回测该缺口并在 CE（50% 中位）附近出现拒绝 K 线时，若同时满足至少 minConf 层方向一致的汇合（趋势 EMA、动量、量能、VWAP 位置、ADX 方向），则顺缺口方向入场的期望收益为正；该逻辑可因「缺口被完全回填且无拒绝」「汇合层失效」「低波动 ADX 不足」而证伪。

经济逻辑：机构/知情资金留下的低效定价区间（FVG）常被回补；回测 + 拒绝表示承接，多层趋势/量能同向降低逆势噪音交易概率。

【标的与频率】
- 市场：加密 USDT 永续为主（优先 BTCUSDT / ETHUSDT）；可选 EURUSD
- 频率：先 15m 或 1h（与 Pine 位移/缺口语义更接近）；可扩展 5m
- 数据源约定：https://github.com/JasonleeQAQ/multi-asset-ohlcv （Parquet 月分片；实际 cloud_bundle 需本地挂载）
- 路径约定：DATA_ROOT/coinraw_data/BTCUSDTraw_data/ohlcv/{tf}/BTCUSDT_{tf}_ohlcv_YYYY-MM.parquet

【数据需求】
- OHLCV：open, high, low, close, volume；时间索引 UTC
- 对齐：K 线开盘时间；禁止用未来 bar 的 high/low 判定当前 FVG
- 样本划分建议：训练/研发 ≤ 2023-12；OOS 盲测 2024-01 → 2025-06（可调，但 OOS 禁止参与调参）

【成功标准 AC-】
- AC-1：OOS 期双引擎方向信号一致率 ≥ 约定阈值（默认 90% 同向 bar）或差异可归因
- AC-2：回测计入手续费+滑点后，OOS 不出现「仅靠忽略成本才盈利」
- AC-3：FVG 检测无前视（仅用 bar_index-2 及更早已确认 bar）
- AC-4：风格/制度失效监控：ADX 过低时段信号应被过滤
- AC-5：KILL- 熔断已写死（例：滚动 30 日最大回撤 > 12% 或 实盘信号偏差率连续 5 日 > 8% → 暂停）

【Pine 参数基线（可复现起点，非优化后参数）】
- minGapAtr=0.15, dispMult=1.15, maxFvgAge=120, minConf=3
- EMA 8/21/50；Squeeze BB20×2 / KC20×1.5；momLen=12
- volLen=14, obvLen=10；ADX14 min 18
- SL 1.2 ATR（可选 beyond FVG edge）；TP1 1.8 / TP2 3.5；trail 1.5 ATR
- cooldown=8 bars；session 可按 24/7 加密关闭或保留

【硬性禁止】
- 禁止先看净值曲线再改 H-1 表述
- 禁止 OOS 区间参与 minConf/ATR 倍数网格搜索
- 禁止未过 ReviewerData / ReviewerHypothesis 进入因子实现
- 禁止未写 KILL- 就 Deploy

【待确认 OP-】
- OP-1：本实验默认标的是否仅 BTCUSDT 15m？
- OP-2：cloud_bundle 本地挂载路径？
- OP-3：模拟盘用 OKX 还是仅研究阶段止于回测？

【风险 RISK-】
- RISK-1：Pine 为 indicator 非 strategy，成交假设与实盘差异大
- RISK-2：Volume Delta 用 candle 内近似，非真实主动买卖
- RISK-3：加密 24/7 与 session filter 语义不一致需显式选择
```
