# Multica Best Practices

[English](./README.en.md) | 中文

> 一个复用度极高的超级个体编排流程（从 PRD 到 CICD）。
> 面向 [Multica](https://github.com/multica-ai/multica) 的 Agent · Squad · Skill · Issue 实战模板。
> **Copy. Paste. Run.** —— 复制即用；或者一条命令自动建好整套。

本仓库把「一个需求从 Issue 走到可上线」需要的**角色、流程、门禁、平台对接**做成可直接复制的配置。你不必从零写 Prompt：复制一个 Starter → 在平台壳里填 `.env` → 跑真实任务，再按团队情况裁剪。

> **背景**：很多团队在 Multica 上反复重造「需求 → 设计 → 实现 → 测试 → 部署」的轮子，还把 Confluence / Jira / Jenkins / Figma 等平台地址与凭据写死在 Prompt 里——换一家公司或平台就得重写。
> 本仓库的核心解法：**平台只出现在 `multica-platform-*` 占位壳，角色提示词只说「用哪个 skill」**；Skill 按名字挂载，团队填壳即复用。所有模板都经过真实任务验证，并公开为「Copy. Paste. Run.」。

![Multica Best Practices 介绍](./display.png)

## 这是什么

一句话：**一套面向真实任务持续迭代的 Multica 小队配置**——每个 Agent 只负责一件事，Leader 负责编排与门禁，每步产出都要证据。

```text
你创建：Agent（角色） + Squad（编排） + Skill（做法） + Issue（任务）
                  ↓
       Leader 带队：需求收敛(G0) → 设计 → 实现 → 单测 → 部署 → 自动化测试
                  ↓
   每步门禁（multica-verification skill 复跑）→ Human 最终验收
```

## 角色与职责（Agent Matrix）

| Agent | 该做 | 不该做 |
| --- | --- | --- |
| Architect | 设计方案 | 大量写代码 |
| FrontendDev | 前端实现（对接 UI 设计） | 修改需求 / 自审自放行 / 发明 API |
| BackendDev | 后端实现 + API 契约 | 修改需求 / 自审自放行 / 处理 UI |
| Tester | T1/T2 用例与覆盖率 / T3 部署后自动化验证 | 修改需求 / 在 G2.5 前跑自动化 |
| DevOps | G2 后触发 CI/CD、回传部署 URL | 写业务代码 / 自宣部署成功 |
| Reviewer | 业务评审（设计 / 关键改动） | 替代客观验证 / 替代人类验收 |
| Leader | 编排与门禁（用 multica-verification skill） | 亲自实现 / 给自己盖章 |

> 注：`software-development-reviewed` Starter 在上述角色之外，为除 Leader、DevOps 外的每个常规产出角色配备了专属 Reviewer。量化研究工厂见下方 `quant-research-factory`。

## Starters

| Starter | 用途 | 状态 |
| --- | --- | --- |
| [Software Development (Reviewed)](./templates/zh_CN/squad/software-development-reviewed) | 推荐主流程：每个常规角色配专属 Reviewer，两层门禁（通用门禁 + 专业产出物评审） | 推荐 |
| [Software Development](./templates/zh_CN/squad/software-development) | 轻量替代：前后端按范围路由，任意角色可缺失，仅一层通用门禁（无专业评审） | 可选 |
| [Bug Fix](./templates/zh_CN/squad/bug-fix) | 根因 / 修复 / 回归（按影响面路由，跳过 Architect） | 实验性 |
| **[Quant Research Factory](./templates/zh_CN/squad/quant-research-factory)** | **量化策略七步研究链路（视频对齐）+ 每步独立审核员 + 3 巡查员 + 书记员；RD-Agent / Qlib / Nautilus / VibeShell** | **新增** |

更多 Starter 将基于真实任务验证后补充。**不要假装最佳实践已经完成。**

## 仓库结构

```text
AGENTS.md     ⭐ Agent 入口：项目约定与改动规范
templates/  ⭐ 从这里开始：可直接复制的全部配置
├── zh_CN/              中文模板（默认；复制整个子目录即用）
│   ├── agents/           共享 Agent Instructions（含 quant-* 量化角色）
│   ├── skills/           共享 Skill
│   └── squad/            小队 Starter
│       ├── software-development/
│       ├── software-development-reviewed/
│       ├── bug-fix/
│       └── quant-research-factory/   ← 量化研究工厂
└── en_US/              英文模板
docs/
scripts/multica-sync/
```

## Quant Research Factory 速览

严格对齐《一条量化策略到底是怎样做出来的？》七步：数据 → 假设 → 因子 → 模型 → 回测 → 执行 → 复盘下线。

- 每步后独立审核员 + 详细 checklist，错误/幻觉立即打回。
- 巡查员 A(1-2-3) / B(4-5-6) / C(7+部署)：主动学习官方文档与案例，解救卡住>3分钟或被打回的 Agent。
- 书记员：全程记录卡点与错误，供人工优化工作流。
- 技术约束：RD-Agent + Qlib + XGBoost + 双引擎（RD-Agent ↔ Nautilus）+ Credal + VibeShell + 20 模拟账户。

详见 [templates/zh_CN/squad/quant-research-factory](./templates/zh_CN/squad/quant-research-factory)。

## 其余文档与流程

软件开发主流程、门禁、命名、平台协作等仍见原 `docs/zh_CN/` 与上方 software-development 相关章节（本 README 精简保留核心入口；完整软件开发流程图见 git 历史或上游 it235/multica-best-practices）。

## License

与上游一致，见 [LICENSE](./LICENSE)。

---

Fork 自 [it235/multica-best-practices](https://github.com/it235/multica-best-practices)，新增 quant-research-factory 量化研究工厂模板。
