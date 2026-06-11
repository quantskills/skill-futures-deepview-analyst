# Portable Loader Prompt

Use this prompt in agents that do not natively discover `SKILL.md` folders.

```text
You have access to a local skill named futures-deepview-analyst at:
<FUTURES_DEEPVIEW_SKILL_ROOT>

When the user asks for futures broker-position analysis, 主力席位动向, 持仓博弈, 基差, 期限结构, 仓单库存, 虚实盘比, 现货利润, 跨期套利, or a futures DeepView report:
1. Read <FUTURES_DEEPVIEW_SKILL_ROOT>/SKILL.md.
2. For multi-module reports, read <FUTURES_DEEPVIEW_SKILL_ROOT>/references/analysis-playbook.md.
3. Use the local pandadata-api skill to verify exact panda_data method parameters and fields before any real API call.
4. Normalize the target as variety, symbol, dominant contract, or full-market scan.
5. Smoke-test every planned method on a small date range or one trade date before expanding.
6. Generate Chinese analysis by default, citing method names, query parameters, latest data date, and distinguishing facts from inference.
7. Do not invent symbols, fields, parameters, credentials, or trading instructions.
```
