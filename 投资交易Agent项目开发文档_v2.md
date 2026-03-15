# AI 投资交易 Agent 项目开发文档

> Version 2.0 | 2026.03

---

## 一、项目定位

构建一套 AI 驱动的投资交易 Agent 系统，覆盖从**信息抓取 → 价值研判 → 智能推送 → 深度研究 → 交易决策 → 执行监控**的完整投资链路。

**第一优先级：信息研判与推送。** 系统的核心起点不是"分析"，而是在海量信息中快速识别什么值得关注、为什么值得关注、对投资有什么影响，并将结论主动推送给用户。

**核心架构选择：Multi-Agent + Skill 混合体系**

- **Agent** 负责决策流转（动态、有状态、可主动触发）
- **Skill** 作为 Agent 内部的标准化工具模块（静态、无状态、可复用）

---

## 二、投资逻辑链

```
信息抓取 → 价值研判 → 智能推送 → 多维分析 → 投资决策 → 技术分析 → 交易计划 → 执行监控
|______ 信息情报阶段 ______|  |____ 投研阶段 ____|  |________ 交易阶段 ________|
```

### 2.1 信息情报阶段（本项目第一优先级）

这是整个系统的「入口」和「过滤器」，解决投资中最基础也最关键的问题：**什么信息值得看？**

**流程拆解：**

```
全量信息抓取 → 去重清洗 → 价值评分 → 重要性分级 → 核心摘要生成 → 投资影响分析 → 推送给用户
```

**信息价值研判的四个维度：**

| 维度 | 判断标准 | 举例 |
|------|---------|------|
| **时效性** | 是否是刚发生的、市场尚未充分定价的信息 | 央行突然降息 vs 一周前的旧闻 |
| **影响面** | 影响的是个股、板块还是整个市场 | 单公司财报 vs 全行业监管政策 |
| **确定性** | 是已确认事实还是市场传闻 | 正式公告 vs 小道消息 |
| **关联度** | 与用户持仓/关注标的的相关程度 | 持仓股的供应商出事 vs 无关行业新闻 |

**推送输出标准格式：**

每条推送必须包含三个部分——

1. **发生了什么**（一句话事实摘要）
2. **为什么重要**（核心分析：影响哪些板块/个股、影响路径是什么）
3. **投资建议**（建议关注/建议加仓/建议减仓/暂无操作，附简要理由）

### 2.2 投研阶段

在信息情报的基础上，对值得深入研究的标的进行系统性分析。

**输出「投资决策」**= 买什么 + 为什么买 + 核心假设 + 失效条件

### 2.3 交易阶段

基于投资决策，叠加技术面和资金面分析，制定可执行的交易计划。

**输出「交易计划」**= 买入价 + 仓位 + 目标价 + 止损线 + 加减仓规则

三个阶段通过明确的接口衔接：信息研判 → 触发投研 → 投资决策 → 驱动交易。

---

## 三、五 Agent 协作架构

在原有四 Agent 基础上，新增 **Intel Agent（信息研判 Agent）** 作为系统入口。

| Agent | 职责 | 输出 |
|-------|------|------|
| **Intel Agent** 🔥 | 信息抓取、价值研判、重要性分级、智能推送 | 分级情报推送（事实 + 分析 + 建议） |
| **Macro Agent** | 宏观经济 + 行业趋势研判 | 市场环境评估（风险等级、板块轮动信号） |
| **Research Agent** | 公司深度研究 | 投资决策（标的 + 方向 + 核心假设） |
| **Trading Agent** | 技术分析 + 交易计划制定 | 五要素交易计划 |
| **Executor Agent** | 执行 + 监控 + 反馈 | 执行日志 + 异常预警 |

**数据流向：**

```
Intel Agent ──→ 用户（实时推送）
    |
    ├─ 重要性高 → Macro Agent → Research Agent → Trading Agent → Executor Agent
    |                                ↑                                  |
    |                                └─── 反馈回路（假设失效/止损）──────┘
    |
    └─ 重要性低 → 归档存储（供后续检索）
```

**Intel Agent 是唯一直接面向用户的推送入口。** 其他 Agent 的分析结果也通过 Intel Agent 整合后统一推送，避免信息碎片化。

### Intel Agent 工作模式

| 模式 | 触发条件 | 推送方式 |
|------|---------|---------|
| **实时预警** | 突发重大事件（政策变动、黑天鹅、异动等） | 即时推送，标记为「紧急」 |
| **定时简报** | 每日早盘前 / 收盘后 | 结构化日报（市场概览 + 重点事件 + 持仓影响） |
| **关联推送** | 与用户持仓/关注列表相关的新信息 | 按关联度打分，超过阈值即推送 |
| **深度触发** | 信息研判后认为值得深入分析 | 自动触发 Research Agent 启动深度研究 |

---

## 四、Skill 规划总表

### 4.1 Intel Agent Skills 🔥（第一优先级）

| Skill | 功能 | 优先级 |
|-------|------|--------|
| **info-crawler** | 多源信息抓取：财经新闻、公告、研报摘要、社交媒体（雪球等）、监管动态 | P0 |
| **info-dedup-cleaner** | 信息去重、清洗、标准化（同一事件多源报道合并为一条） | P0 |
| **info-value-scorer** | 信息价值评分：按时效性、影响面、确定性、关联度四维打分，输出重要性等级（S/A/B/C） | P0 |
| **insight-generator** | 核心分析生成：一句话摘要 + 影响路径分析 + 投资建议，形成标准推送卡片 | P0 |
| **push-dispatcher** | 推送分发：根据重要性等级和用户偏好，决定推送方式（即时/定时/归档） | P0 |
| **daily-briefing-builder** | 每日投资简报生成：整合当日所有情报，按板块/持仓/宏观分类，生成结构化日报 | P0 |
| **watchlist-matcher** | 关注列表匹配：将新信息与用户持仓和关注标的做关联匹配，计算相关性分数 | P1 |

### 4.2 Macro Agent Skills

| Skill | 功能 | 优先级 |
|-------|------|--------|
| macro-data-collector | 采集宏观指标（GDP、CPI、PMI、利率、汇率、大宗商品） | P0 |
| policy-analyzer | 解析央行声明、政策文件，提取方向性信号 | P0 |
| geopolitical-monitor | 追踪地缘事件，评估市场冲击等级 | P1 |
| sector-rotation-detector | 基于资金流向和宏观周期识别板块轮动 | P1 |

### 4.3 Research Agent Skills

| Skill | 功能 | 优先级 | 复用 |
|-------|------|--------|------|
| financial-report-parser | 提取年报/季报结构化数据 | P0 | ✅ 复用 xlsx + pdf Skill |
| business-model-analyzer | 商业模式、护城河、增长驱动力分析 | P0 | 新建 |
| valuation-engine | DCF / 可比公司 / 可比交易估值 | P0 | 新建 |
| industry-chain-mapper | 上下游产业链映射 + 议价能力分析 | P1 | 新建 |
| competitor-benchmarker | 同行关键指标对比 | P1 | ✅ 复用 xlsx Skill |
| research-report-generator | 生成结构化投研报告 | P1 | ✅ 复用 docx Skill |

### 4.4 Trading Agent Skills

| Skill | 功能 | 优先级 |
|-------|------|--------|
| technical-indicator-calc | 计算 MA/MACD/RSI/布林带/支撑阻力位 | P0 |
| position-sizer | 基于风险预算和波动率计算最优仓位 | P0 |
| trading-plan-generator | 生成完整五要素交易计划 | P0 |
| price-pattern-recognizer | 识别头肩顶/双底/突破等图形形态 | P1 |
| fund-flow-analyzer | 北向资金、两融数据、主力资金流向追踪 | P1 |

### 4.5 Executor Agent Skills

| Skill | 功能 | 优先级 | 复用 |
|-------|------|--------|------|
| hypothesis-tracker | 监控核心假设是否仍然成立 | P0 | 新建 |
| alert-engine | 价格/成交量/新闻异常预警 | P0 | 新建 |
| order-executor | 按交易计划下单执行 | P0 | 新建（需对接券商API） |
| portfolio-monitor | 实时盈亏、仓位集中度、回撤监控 | P1 | 新建 |
| trade-journal | 自动记录交易日志，生成复盘报告 | P1 | ✅ 复用 docx + xlsx Skill |

### 4.6 跨 Agent 共享 Skills

| Skill | 功能 | 调用方 |
|-------|------|--------|
| data-fetcher | 统一行情数据接口（A股/港股/实时/历史），建议做成 MCP Server | 全部 Agent |
| llm-summarizer | 长文本摘要（研报/新闻/会议纪要） | 全部 Agent |
| chart-visualizer | 生成K线图/对比图/持仓热力图 | Trading / Executor |

> **注意**：原「news-aggregator」已被 Intel Agent 的 info-crawler + info-dedup-cleaner 替代，新闻采集统一归口到 Intel Agent。

---

## 五、可复用的现有 Skill

| 现有 Skill | 复用场景 |
|-----------|---------|
| **xlsx**（电子表格） | 财报数据提取、估值模型、同业对比表、持仓跟踪表 |
| **docx**（Word 文档） | 投研报告、交易日志、策略备忘录、每日简报（长版） |
| **pdf**（PDF 处理） | 读取年报、招股书、监管文件 |
| **frontend-design** | 交互式仪表板、K线图、推送卡片 UI、实时监控界面 |
| **pptx**（演示文稿） | 投委会路演材料（按需） |

> 共 26 个 Skill：**5 个可直接复用**现有平台 Skill，**21 个需新建**（其中 Intel Agent 占 7 个）。

---

## 六、技术选型

| 层级 | 技术 | 说明 |
|------|------|------|
| Agent 编排 | Claude API（tool_use） | 原生工具调用 + 强推理能力 |
| 数据层 | MCP Server（TypeScript） | 统一数据接口，无状态可扩展 |
| 后端 | FastAPI（Python） | 量化/金融库生态丰富（pandas, talib） |
| 前端 | Next.js + React | 与 Crypto Radar 技术栈一致，可复用组件 |
| 数据库 | PostgreSQL + TimescaleDB | 时序数据优化（K线、交易日志） |
| 缓存 | Redis | 实时行情缓存、预警状态、推送队列 |
| 消息推送 | SSE + WebSocket | 实时情报推送到客户端 |
| 定时任务 | Celery / Cron | 定时抓取、每日简报生成 |

**与 Crypto Radar 的复用点**：Multi-Agent 编排框架、SSE 推送、信号执行循环、前后端技术栈均可直接迁移。

---

## 七、开发节奏

### Phase 1：信息情报 MVP（第 1-4 周）🔥

**目标**：跑通「信息抓取 → 价值研判 → 推送给用户」的核心闭环。

- 搭建 data-fetcher MCP Server（A股行情 + 新闻源接入）
- 实现 Intel Agent 核心链路：
  - info-crawler（接入 3-5 个核心信息源：财联社/东方财富/巨潮资讯/雪球/央行官网）
  - info-dedup-cleaner（基础去重和清洗）
  - info-value-scorer（四维评分模型，先用规则 + LLM 混合方案）
  - insight-generator（生成标准推送卡片：事实 + 分析 + 建议）
  - push-dispatcher（基础推送逻辑）
- 实现 daily-briefing-builder（每日早报/晚报）
- 搭建前端推送卡片 UI（复用 frontend-design）
- 端到端验证：系统自动抓取信息 → 筛选出重要信息 → 生成分析推送 → 用户在终端收到

### Phase 2：投研能力（第 5-8 周）

**目标**：在情报推送基础上，增加深度研究和交易计划能力。

- 实现 Research Agent + financial-report-parser + valuation-engine + business-model-analyzer
- 实现 Trading Agent + technical-indicator-calc + trading-plan-generator + position-sizer
- 定义 Agent 间的接口规范（投资决策格式）
- 实现 Intel Agent → Research Agent 的自动触发（info-value-scorer 判定为 S 级时自动启动深度研究）
- 实现 watchlist-matcher（关注列表关联匹配）
- 端到端验证：重大信息触发 → 自动深度研究 → 输出投资决策 + 交易计划

### Phase 3：执行闭环 + 增强智能（第 9-12 周）

**目标**：全链路闭环 + 高级分析 + 复盘体系。

- 实现 Macro Agent + macro-data-collector + policy-analyzer
- 实现 Executor Agent + hypothesis-tracker + alert-engine + portfolio-monitor
- 实现反馈回路：Executor → Research 重新评估触发
- 实现 sector-rotation-detector、fund-flow-analyzer
- 搭建交互式仪表板（复用 frontend-design）
- 实现 trade-journal 自动复盘 + research-report-generator（复用 docx）
- 用户测试与迭代优化

---

## 八、关键风险

| 风险 | 应对 |
|------|------|
| LLM 幻觉导致财务数据错误 | 所有数据必须来自 API，禁止 LLM 凭记忆生成数字 |
| 信息源质量参差不齐 | info-value-scorer 标记信息源可信度；引入多源交叉验证 |
| 推送过多导致信息疲劳 | 严格分级机制：S 级即时推送、A 级汇总推送、B/C 级仅归档 |
| API 限流 / 数据成本 | Redis 缓存 + 批量查询 + 免费数据源兜底 |
| Agent 间协调断裂 | 明确接口规范；每个 Agent 可独立运行、接受手动输入 |
| A股合规风险 | 初期不做自动下单，所有决策需人工确认；保留完整审计轨迹 |

---

## 九、成功指标

| 指标 | 目标 |
|------|------|
| 重要信息识别准确率（S/A 级） | > 80% |
| 信息抓取到推送延迟 | < 2 分钟 |
| 每日简报用户打开率 | > 70% |
| 端到端延迟（股票代码 → 交易计划） | < 3 分钟 |
| 研究假设命中率 | 6 个月后 > 60% |
| 交易计划执行合规率 | > 90% |
| 每日简报覆盖度 | Top 10 板块 + 持仓个股 |
