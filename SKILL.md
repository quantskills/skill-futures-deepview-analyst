---
name: futures-deepview-analyst
description: "基于 Pandadata 期货 DeepView 接口做期货品种、主力合约和跨期结构的综合研判，覆盖席位净持仓保证金、席位建仓、盈亏排行、多空比、资金流、基差、期限结构、仓单库存、虚实盘比、现货利润和跨期套利。Use when the user asks for 期货席位分析、持仓博弈、主力席位动向、基差或期限结构监控、仓单库存跟踪、虚实盘比、跨期套利扫描、期货品种综合分析, or a futures DeepView report for a variety or contract."
license: GPL-3.0-only
---

# Futures DeepView Analyst

将“分析螺纹钢席位博弈”“看豆粕期限结构和仓单”“扫描哪些品种有跨期套利机会”这类请求，转成 Pandadata 期货 DeepView 数据调用计划，并输出结构化中文研判报告。

## Core Rules

- Use the `pandadata-api` skill for all API contract checks and real data calls. Open `references/method-index.md` or run `scripts/search_api_docs.py --method <method>` before using any method.
- Do not invent parameters, fields, symbols, exchange suffixes, or credentials. Use `get_future_detail` and `get_future_dominant` to confirm available variety/contract codes when the user's input is informal.
- Separate facts from interpretation. State data facts first, then label directional conclusions as inference and show the evidence chain.
- Prefer a compact answer when the user asks for a quick view. Produce a full report only when the user asks for 深度分析、完整报告、HTML/Markdown 输出、扫描, or when the request spans multiple modules.

## Workflow

1. **Normalize the target**: identify whether the request is for a variety, a specific symbol, a dominant contract, or a market-wide scan. Default date window: latest available trade date for snapshots; recent 20 trading days for trend checks; recent 120 trading days for percentile context.
2. **Confirm the contract universe**: call `get_future_detail` for contract metadata and `get_future_dominant` for dominant contract mapping. Record the final variety/symbol/date parameters in the report.
3. **Build a data plan by topic**: choose only the method groups needed for the user's question. Use the interface map below, then load exact method docs through `pandadata-api`.
4. **Smoke-test each method**: run one small call first, inspect `head()`, `shape`, column names, date coverage, and empty-data causes before expanding the date range or universe.
5. **Analyze with same-date alignment**: align position, basis, term-structure, inventory, and arbitrage snapshots on the same `trade_date` whenever drawing cross-module conclusions.
6. **Generate the deliverable**: cite method names, query parameters, and data截止日 for every chart/table/conclusion. End with confidence level, key risks, and data limitations.

## Interface Map

| Question | Primary methods |
|---|---|
| 基础行情与合约确认 | `get_future_daily`, `get_future_daily_post`, `get_future_min`, `get_future_detail`, `get_future_dominant` |
| 席位持仓、建仓和净多空 | `get_broker_netmarg`, `get_broker_netmarg_change`, `get_broker_totlmarg`, `get_broker_grade`, `get_broker_oi_value`, `get_broker_build_process`, `get_future_nonbroker_net`, `get_future_netposi_rank`, `get_future_netcap_change` |
| 席位盈亏和资金流 | `get_broker_profit`, `get_broker_variety_profit`, `get_broker_profit_rank`, `get_broker_loss_rank`, `get_broker_flow_daily` |
| 多空情绪和合约排行 | `get_future_ls_ratio`, `get_broker_ls_ratio`, `get_future_contract_indicators`, `get_future_contract_rank`, `get_future_contract_pool`, `get_future_net_flow` |
| 基差、期限结构和相关性 | `get_future_basis`, `get_future_term_structure`, `get_future_dominant_corr` |
| 仓单、库存、虚实盘和现货利润 | `get_future_warehouse_receipt`, `get_future_inventory`, `get_future_virtual_ratio`, `get_future_trader_quote`, `get_future_spot_profit` |
| 跨期套利和价差价比 | `get_future_calendar_arbitrage`, `get_future_free_spread`, `get_future_free_ratio` |
| 持仓规模和市值 | `get_future_variety_posi`, `get_future_symbol_posi`, `get_future_variety_mcap` |

## Analysis Modes

- **席位博弈**: compare price trend with net position margin, net capital change, build process, broker grade, profit/loss ranks, and top net-position lists. Flag divergence only when price, position, and capital evidence point in different directions.
- **结构研判**: combine basis, term structure, dominant correlation, calendar arbitrage, free spread, and free ratio. Use one trade date for cross-contract comparisons.
- **库存现货**: combine warehouse receipt, inventory, virtual ratio, trader quote, and spot profit to explain deliverable pressure, squeeze risk, or cash-market support.
- **全品种扫描**: rank candidates first, then drill into the top names. Keep scan output tabular and show the filters used.

For detailed signal recipes, report templates, and empty-data handling, read `references/analysis-playbook.md`.

## Output Standards

- Write reports in Chinese unless the user requests another language.
- Use Markdown by default. Use HTML only when the user asks for a webpage/report artifact.
- Keep numerical precision practical: prices and ratios use market conventions; amounts must state units such as 元、万元、亿元, or 保证金/市值口径.
- Every substantive conclusion must include: source method, target variety/symbol, query dates, latest data date, and whether the conclusion is fact or inference.
- Do not issue trading instructions. If the user asks to turn the analysis into strategy code or orders, hand off to `ssquant-ai-trader` or `ssquant-trader-generator` after the data conclusions are clear.

## Data Caveats

- Broker-position data is limited by exchange disclosure rules, such as top-rank coverage. State that limitation when interpreting “主力席位”.
- DeepView methods often return large single-day snapshots. Validate on one date before broad pulls.
- Empty results are not automatically service failures. Check date availability, symbol/variety format, required filters, and whether the market was open.
- Avoid stitching different trade dates into one signal unless the report explicitly labels it as asynchronous evidence.
