# Futures DeepView Analysis Playbook

Load this file when a request needs more than a quick single-topic answer, when comparing multiple DeepView modules, or when producing a formal Markdown/HTML report.

## Request Triage

Classify the user's request before calling data:

| User intent | Minimum modules | Useful expansion |
|---|---|---|
| “席位怎么看”“主力是否看多” | 行情、主力合约、席位持仓 | 盈亏排行、建仓过程、资金流、多空比 |
| “基差/期限结构” | 行情、基差、期限结构 | 跨期套利、主力相关性、库存 |
| “仓单/库存压力” | 行情、仓单、库存 | 虚实盘比、现货报价、现货利润 |
| “跨期套利机会” | 期限结构、跨期套利、自由价差/价比 | 成交持仓、主力合约、库存变化 |
| “全品种扫描” | 排行或池子类接口 | 对排名靠前品种做二级验证 |

If the user omits a date, use the latest available `trade_date` found from the first successful method call. If the user omits a lookback window, use 20 trading days for near-term behavior and 120 trading days for percentile context.

## Data Calling Pattern

1. Use `pandadata-api` to inspect exact docs for every method in the planned call list.
2. Run a tiny call first: one variety/symbol and one date or a short date range.
3. Log the observed columns and date coverage. Only compute ratios after checking units and field meanings from the method docs.
4. For broad scans, apply the broad/rank method first, then use detail methods only for the top candidates.

Preferred validation notes to include in the working analysis:

- `method`: exact Pandadata method name.
- `params`: key filters, especially variety/symbol/date.
- `rows`: row count after filters.
- `latest_date`: max trade date in returned data.
- `empty_reason`: if empty, document the next check performed.

## Signal Recipes

### Broker Position Game

Use `get_broker_netmarg`, `get_broker_netmarg_change`, `get_broker_totlmarg`, `get_future_netposi_rank`, and `get_broker_build_process`.

Compute:

- Net direction: long-side margin minus short-side margin, or the documented net field.
- Intensity: net change divided by total open-interest margin or recent average when a compatible denominator exists.
- Confirmation: price change, open interest change, and net position change moving in the same direction.
- Divergence: price rises while net long margin weakens, or price falls while net long margin strengthens.
- Concentration: top brokers' net exposure share when ranking fields support it.

Use cautious wording. “净多增加” is a fact; “多头更主动” is an inference; “必然上涨” is not allowed.

### Broker Profit And Flow

Use `get_broker_profit`, `get_broker_variety_profit`, `get_broker_profit_rank`, `get_broker_loss_rank`, and `get_broker_flow_daily`.

Look for:

- Whether profitable brokers are increasing or reducing exposure.
- Whether loss-making brokers are forced to cut positions.
- Whether large daily fund flow agrees with net-position direction.
- Whether one broker dominates the signal or the move is broad-based.

### Basis And Term Structure

Use `get_future_basis`, `get_future_term_structure`, `get_future_calendar_arbitrage`, `get_future_free_spread`, and `get_future_free_ratio`.

Compute:

- Basis and basis rate using documented fields.
- Percentile of basis over the chosen lookback window.
- Curve shape: contango, backwardation, local inversion, or flattening/steepening.
- Calendar spread change on the same trade date as the term-structure snapshot.

Never compare contracts from different trade dates without explicitly labeling the comparison as asynchronous.

### Inventory, Warehouse Receipts, And Cash Market

Use `get_future_warehouse_receipt`, `get_future_inventory`, `get_future_virtual_ratio`, `get_future_trader_quote`, and `get_future_spot_profit`.

Analyze:

- Warehouse receipt and inventory direction: day-over-day, week-over-week, and year-over-year if enough history exists.
- Virtual/physical pressure: explain high or rising virtual ratio with the exact denominator used by the method docs.
- Spot profit: production margin expansion or compression, and whether it supports supply growth or cuts.
- Cash quote consistency: whether trader quotes support the futures premium/discount story.

### Full-Market Scan

For scans, avoid pulling every detail table for every contract at the start.

Suggested order:

1. Use rank/pool methods such as `get_future_contract_rank`, `get_future_contract_pool`, `get_future_net_flow`, `get_future_netcap_change`, or `get_future_calendar_arbitrage`.
2. Define filters visibly: liquidity, rank threshold, change threshold, same-date availability.
3. Select top 5-10 candidates.
4. Run detail validation for those candidates only.
5. Return a table with candidate, signal, methods used, latest date, and follow-up check.

## Report Skeleton

Use this structure for full reports:

1. **摘要结论**: 3-5 bullets with direction, confidence, and biggest contradiction.
2. **数据范围**: variety/symbol, date window, latest data date, methods used.
3. **行情与合约背景**: price trend, dominant contract, liquidity/open interest context.
4. **席位博弈**: net position, build process, broker profit/loss, flows.
5. **基差与期限结构**: basis percentile, curve shape, calendar spread/arbitrage signals.
6. **仓单库存与现货**: warehouse receipt, inventory, virtual ratio, spot profit, trader quote.
7. **交叉验证**: where modules agree, where they conflict, and what could invalidate the view.
8. **风险与限制**: disclosure limitations, empty data, stale dates, contract rollover, liquidity.
9. **附录**: method/parameter table and row counts.

For a quick answer, compress the skeleton to: 结论, 关键证据, 矛盾点, 数据来源.

## Empty Or Conflicting Data

When a method returns empty:

1. Check whether the date is a trading day and whether the method expects `trade_date`, `start_date/end_date`, or another date field.
2. Check whether the method expects variety code, symbol code, exchange suffix, or contract month.
3. Try a known dominant contract from `get_future_dominant`.
4. Reduce optional filters and request all fields if the method docs allow it.
5. Report “该接口在所选条件下无返回” with the tested method and parameters.

When modules conflict:

- Do not force a single conclusion. State which evidence is price-based, position-based, cash-market-based, or arbitrage-based.
- Lower confidence when price, position, and cash-market indicators disagree.
- Highlight the next data point that would resolve the conflict, such as next trade date broker positions or updated warehouse receipts.
