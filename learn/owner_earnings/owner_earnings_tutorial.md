# Owner Earnings（所有者盈余）估值教程

> 本教程使用 [ValueInvest](https://github.com/wangzhe3224/valueinvest) 库，带你从零理解巴菲特最看重的盈利质量指标——Owner Earnings，并用真实数据进行实战分析。

## 学习目标

1. 理解 Owner Earnings 的概念和巴菲特的原始定义
2. 掌握公式中每个组成部分的经济含义
3. 区分「净利润」和「所有者盈余」的差异
4. 使用 `valueinvest` 库获取真实数据并估值
5. 手动计算每一步，理解盈利质量评估
6. 多公司对比，识别盈利质量差异

---

## 1. 什么是 Owner Earnings？

### 巴菲特的原始定义（1986 年致股东信）

> "If we think through these questions, we can gain some insights about what may be called 'owner earnings.' These represent (a) reported earnings plus (b) depreciation, depletion, amortization, and certain other non-cash charges... less (c) the average annual amount of capitalized expenditures for plant and equipment, etc. that the business requires to fully maintain its long-term competitive position and its unit volume."

翻译过来就是：**所有者盈余** = 报告利润 + 非现金费用 - 维持竞争力的资本支出 - 营运资金变动

### 核心公式

$$\text{Owner Earnings} = \text{Net Income} + \text{D\&A} - \text{Maintenance CapEx} - \Delta\text{NWC}$$

| 组成部分 | 含义 | 为什么重要 |
|----------|------|-----------|
| **Net Income** | 报告净利润（起点） | 这是利润表的"底线数字"，但并不代表公司实际赚到的现金 |
| **+ Depreciation & Amortization** | 加回非现金折旧摊销 | D&A 是会计上的费用，但并没有实际现金流出——钱早在买设备时就付了 |
| **- Maintenance CapEx** | 减去维持竞争力所需资本支出 | 公司必须持续投入才能保持竞争力，这部分钱是"必须花"的 |
| **- Change in NWC** | 减去营运资金变动 | 库存和应收账款的增加意味着现金被占用，不能分配给股东 |

### 为什么净利润不够？

净利润（Net Income）的问题是它不等于现金。举个简单的例子：

> 假设一家公司净利润 \$1 亿，但其中 \$3000 万是客户还没付款的应收账款（还没收到现金），同时公司还需要花 \$2000 万更新设备才能维持运营。那么真正能分给股东的钱只有 \$5000 万左右。

具体来说，净利润有三大缺陷：

1. **包含非现金收入** — 可能有大量应收账款未回收，利润记上了但钱没收进来
2. **未扣除维持性资本支出** — 公司必须持续投入才能保持竞争力，这些钱是"刚性支出"
3. **忽略营运资金变化** — 库存积压和应收账款增加都在消耗现金

Owner Earnings 回答的核心问题是：**公司真正能放进股东口袋里的钱是多少？**

### 盈利质量指标

$$\text{Earnings Quality} = \frac{\text{Owner Earnings}}{\text{Net Income}}$$

| 比值 | 含义 | 解读 |
|------|------|------|
| > 1.2 | **Cash-rich** | 实际现金流远超报告利润，盈利质量很高，公司现金生成能力强 |
| 0.8 - 1.2 | **Normal** | 利润与现金基本匹配，盈利质量正常 |
| < 0.8 | **Accrual-heavy** | 报告利润含水量高，需要警惕——可能存在收入虚增或资本支出不足 |

> **实战提示**：Earnings Quality < 0.8 不一定意味着公司在造假，但至少说明利润的"现金含量"不高，值得进一步调查原因。

---

## 2. 获取真实股票数据

Owner Earnings 的计算需要以下数据：

- **Net Income**（净利润）— 来自利润表
- **Depreciation & Amortization**（折旧摊销）— 来自现金流量表
- **CapEx**（资本支出）— 来自现金流量表的投资活动
- **Net Working Capital**（净营运资金）— 来自资产负债表
- **Shares Outstanding**（流通股数）— 用于计算每股价值
- **Cost of Capital**（资本成本）— 用于折现计算
- **Current Price**（当前股价）— 用于判断溢价/折价

我们选择 **Apple (AAPL)** 作为示例——巴菲特的重仓股，也是盈利质量分析的典型案例。

```python
from valueinvest import Stock, ValuationEngine

ticker = "AAPL"
stock = Stock.from_api(ticker)

print(f"公司：{stock.name} ({stock.ticker})")
print(f"当前股价：\${stock.current_price:.2f}")
print(f"净利润 (Net Income)：\${stock.net_income/1e9:.2f}B")
print(f"折旧摊销 (D&A)：\${stock.depreciation/1e9:.2f}B")
print(f"资本支出 (CapEx)：\${stock.capex/1e9:.2f}B")
print(f"净营运资金 (NWC)：\${stock.net_working_capital/1e9:.2f}B")
print(f"流通股数：{stock.shares_outstanding/1e9:.2f}B")
print(f"资本成本：{stock.cost_of_capital:.2f}%")
print(f"增长率：{stock.growth_rate:.1f}%")
```

运行结果：

```
公司：Apple Inc. (AAPL)
当前股价：$246.63

净利润 (Net Income)：$112.01B
折旧摊销 (D&A)：$11.70B
资本支出 (CapEx)：$12.71B
净营运资金 (NWC)：$-17.67B
流通股数：14.68B
资本成本：10.00%
增长率：15.7%
营收：$416.16B
```

> **数据观察**：Apple 的净营运资金为负（\$-17.67B），这在强势消费品牌中很常见——因为 Apple 对供应商有极强的议价能力，应付账款远大于应收账款，相当于"免费使用供应商的钱"。

---

## 3. 理解公式中每个组成部分

让我们一步步拆解 Owner Earnings 公式，理解每一项的经济含义。

### (a) Net Income — 起点

```
Net Income = $112.01B
```

这是利润表上的"底线数字"，代表会计意义上的利润。但正如前面所说，净利润不等于现金，它包含了非现金项目，也没有扣除公司维持竞争力必须花的钱。

### (b) + Depreciation & Amortization — 加回非现金费用

```
D&A = $11.70B
加回后 = $123.71B
```

为什么要加回折旧摊销？因为这是**过去投资的分摊成本**，不是当期的现金流出。举个例子：

> 你花 \$100 万买了一台设备，使用寿命 10 年。每年在利润表上记 \$10 万折旧费用，但实际上这笔钱你在购买时就已经一次性付了。所以在计算"当期真正赚到的现金"时，需要把这笔非现金费用加回来。

### (c) - Maintenance CapEx — 减去维持性资本支出

```
总 CapEx = $12.71B
维持性占比 = 70%
维持性 CapEx = $8.90B
减去后 = $114.81B
```

**这是 Owner Earnings 计算中最关键、也最有争议的一步。**

资本支出（CapEx）可以分为两类：

| 类型 | 含义 | 是否应扣除 |
|------|------|-----------|
| **维持性 CapEx** | 维持现有竞争力和产能的必要投入 | **应该扣除** — 这是"必须花"的钱 |
| **增长性 CapEx** | 用于扩张新业务、新市场的投入 | 不应扣除 — 这是在为未来投资 |

问题在于，公司财报通常只报告**总 CapEx**，不会告诉你多少是维持性的、多少是增长性的。巴菲特本人倾向于保守估计——直接用总 CapEx 代替维持性 CapEx。valueinvest 库默认使用 70% 作为维持性占比。

### (d) - Change in NWC — 减去营运资金变动

```
净营运资金 = $-17.67B
估计变动额 = $1.77B
```

营运资金（Working Capital）= 流动资产 - 流动负债。当公司的库存增加或客户赊账增多（应收账款增加），现金就被"占用"了，不能分配给股东。反之，如果公司能延迟支付供应商（应付账款增加），反而释放了现金。

> **简化说明**：当前实现使用净营运资金绝对值的 10% 作为变动额的代理估计。更精确的做法需要多年历史数据来计算真实的 ΔNWC。

### 最终结果

```
Owner Earnings = 112.01 + 11.70 - 8.90 - 1.77 = $113.04B
vs. Net Income = $112.01B
Earnings Quality = 100.9% (Normal)
```

Apple 的 Owner Earnings 略高于净利润（100.9%），说明盈利质量"正常"——利润和现金基本匹配。折旧摊销加回的效果几乎被维持性 CapEx 抵消了。

---

## 4. 使用 valueinvest 库运行估值

理解了手动计算后，我们用 valueinvest 库一行代码完成计算：

```python
from valueinvest import Stock, ValuationEngine

engine = ValuationEngine()
stock = Stock.from_api("AAPL")
result = engine.run_single(stock, "owner_earnings")
```

输出结果：

```
Owner Earnings 估值结果：Apple Inc. (AAPL)
============================================================

估值结果
   合理价值：$96.25
   当前股价：$246.63
   溢价/折价：-61.0%
   评估：Overvalued

估值区间
   保守：$57.75
   基准：$96.25
   乐观：$105.87

详细信息
   Owner Earnings：$113.04B
   Owner Earnings/Share：$7.70
   盈利质量：100.9%
   隐含 P/E (OE)：32.0x
   零增长价值：$77.00
   含增长价值：$115.50

计算组成
   净利润：$112.01B
   折旧摊销：$11.70B
   维持性CapEx：$8.90B
   营运资金变动：$1.77B

分析要点
   - Owner Earnings: 113.04B (vs Net Income: 112.01B)
   - Owner Earnings/Share: 7.70
   - Earnings Quality: 100.9% (Normal)
   - Implied P/E (Owner Earnings): 32.0x (vs Reported P/E: 31.2x)
   - Zero-growth value: 77.00
   - With 15.7% growth: 115.50
   - Note: Using 10% of NWC as proxy for change in working capital

置信度：Medium
适用性：Applicable
```

### 结果解读

- **合理价值 \$96.25 vs 当前股价 \$246.63**：Owner Earnings 估值显示 Apple 被高估了约 61%
- **隐含 P/E (OE) 32.0x vs 报告 P/E 31.2x**：两者接近，说明 Apple 的盈利质量确实很"实"，没有太多水分
- **估值区间 \$57.75 - \$105.87**：即使在最乐观的假设下，合理价值仍远低于当前股价

> **重要提示**：单一估值方法不能给出完整画面。Owner Earnings 偏保守（尤其是零增长模型），对于 Apple 这样拥有强大品牌和生态壁垒的公司，市场愿意给予更高的溢价。后面的综合分析会结合其他方法。

---

## 5. 手动计算 Owner Earnings（逐步推导）

如果你想知道 valueinvest 库内部是怎么计算的，这里展示了完整的 7 步推导过程：

```python
def manual_owner_earnings(stock):
    shares = stock.shares_outstanding
    revenue = stock.revenue if stock.revenue > 0 else stock.net_income * 10

    # Step 1: 获取净利润
    net_income = stock.net_income
    print(f"Step 1: Net Income = \${net_income/1e9:.2f}B, EPS = \${net_income/shares:.2f}")

    # Step 2: 加回折旧摊销
    dep = stock.depreciation
    if dep == 0:
        dep = revenue * 0.05  # 估算: 5% of revenue
    print(f"Step 2: + D&A = \${dep/1e9:.2f}B, 小计 = \${(net_income + dep)/1e9:.2f}B")

    # Step 3: 减去维持性 CapEx
    capex = stock.capex
    if capex == 0:
        maint_capex = revenue * 0.07  # 估算: 7% of revenue
    else:
        maint_capex = abs(capex) * 0.7
    print(f"Step 3: - Maintenance CapEx = \${maint_capex/1e9:.2f}B")

    # Step 4: 减去营运资金变动
    nwc = stock.net_working_capital
    if nwc != 0:
        nwc_change = abs(nwc) * 0.1
    else:
        nwc_change = revenue * 0.01
    print(f"Step 4: - NWC Change = \${nwc_change/1e9:.2f}B")

    # Step 5: Owner Earnings
    oe = net_income + dep - maint_capex - nwc_change
    oe_per_share = oe / shares
    print(f"Step 5: Owner Earnings = \${oe/1e9:.2f}B, Per Share = \${oe_per_share:.2f}")

    # Step 6: 盈利质量
    eq = oe / net_income if net_income != 0 else 0
    label = "Cash-rich" if eq > 1.2 else ("Accrual-heavy" if eq < 0.8 else "Normal")
    print(f"Step 6: Earnings Quality = {eq:.1%} ({label})")

    # Step 7: 估值
    wacc = stock.cost_of_capital / 100
    g = stock.growth_rate / 100 if stock.growth_rate else 0
    v0 = oe_per_share / wacc
    if wacc > g:
        vg = oe_per_share * (1 + g) / (wacc - g)
    else:
        vg = v0 * 1.5
    fair = (v0 + vg) / 2
    premium = (fair - stock.current_price) / stock.current_price * 100
    print(f"Step 7: Fair Value = \${fair:.2f}, Premium = {premium:+.1f}%")

    return oe
```

输出：

```
Step 1: Net Income = $112.01B, EPS = $7.63
Step 2: + D&A = $11.70B, 小计 = $123.71B
Step 3: - Maintenance CapEx = $8.90B
Step 4: - NWC Change = $1.77B
Step 5: Owner Earnings = $113.04B, Per Share = $7.70
Step 6: Earnings Quality = 100.9% (Normal)
Step 7: Fair Value = $96.25, Premium = -61.0%
```

---

## 6. 可视化 Owner Earnings

![Owner Earnings 组成瀑布图和净利润对比](https://raw.githubusercontent.com/wangzhe3224/valueinvest-learn/main/learn/owner_earnings/plots/owner_earnings_components.png)

**左图解读**：瀑布图展示了从净利润到 Owner Earnings 的调整过程。Apple 的折旧摊销（\$11.7B）几乎完全被维持性 CapEx（\$8.9B）抵消，说明公司的资本支出和资产折旧基本匹配。

**右图解读**：净利润（\$112.0B）和 Owner Earnings（\$113.0B）几乎相等，盈利质量 100.9% 属于"正常"水平。

---

## 7. 多公司盈利质量对比

单一公司的分析说服力有限，让我们对比 5 家知名公司：

```python
tickers = ["AAPL", "MSFT", "GOOGL", "JPM", "BRK-B"]
for t in tickers:
    stock = Stock.from_api(t)
    result = engine.run_single(stock, "owner_earnings")
    # ... 打印结果
```

| Ticker | 公司 | 股价 | 净利润(\$B) | OE(\$B) | OE/股 | EQ | OE合理价 | 折价% | 置信度 |
|--------|------|------|------------|---------|-------|------|---------|-------|--------|
| AAPL | Apple | 246.63 | 112.01 | 113.04 | 7.70 | **1.01** | 96.25 | -61.0% | Medium |
| MSFT | Microsoft | 358.96 | 101.83 | 85.81 | 11.56 | **0.84** | 144.45 | -59.8% | Medium |
| GOOGL | Alphabet | 273.50 | 132.17 | 78.96 | 13.56 | **0.60** | 169.54 | -38.0% | Low |
| JPM | JPMorgan | 283.77 | 57.05 | 51.32 | 19.03 | **0.90** | 225.17 | -20.6% | Medium |
| BRK-B | Berkshire | 474.66 | 66.97 | 61.69 | 44.36 | **0.92** | 427.62 | -9.9% | Medium |

![多公司盈利质量对比](https://raw.githubusercontent.com/wangzhe3224/valueinvest-learn/main/learn/owner_earnings/plots/earnings_quality_comparison.png)

### 关键发现

1. **Apple（EQ = 101%）**：盈利质量正常，利润和现金几乎一一对应
2. **Microsoft（EQ = 84%）**：略低于 1.0，部分利润被资本支出消耗——这与微软在云基础设施和 AI 上的大量投资有关
3. **Alphabet/Google（EQ = 60%）**：盈利质量最低，需要重点关注。Google 的资本支出非常高（数据中心、AI 芯片等），导致 Owner Earnings 远低于净利润
4. **JPMorgan（EQ = 90%）**：银行的 EQ 通常略低于 1.0，因为金融行业有特殊的资本要求
5. **Berkshire（EQ = 92%）**：巴菲特自己的公司，盈利质量稳健

> **深层思考**：Google 的低 EQ 不一定是坏事——它可能正在为未来 AI 基础设施大量投资（增长性 CapEx）。但作为投资者，你需要判断这些投资未来能否产生足够的回报。如果区分了增长性 vs 维持性 CapEx，Google 的真实 EQ 可能会高得多。

---

## 8. 维持性 CapEx 敏感度分析

Owner Earnings 的**最大不确定性**来自于"维持性 CapEx 占总 CapEx 的比例"这个假设。valueinvest 默认使用 70%，但让我们看看不同假设如何影响结果：

![维持性CapEx敏感度分析](https://raw.githubusercontent.com/wangzhe3224/valueinvest-learn/main/learn/owner_earnings/plots/capex_sensitivity.png)

**左图解读**：随着维持性 CapEx 占比从 50% 增加到 100%，Apple 的 Owner Earnings 从约 \$116B 降至约 \$109B。变化幅度约 6%，相对温和。

**右图解读**：即使维持性 CapEx 占比达到 100%（最保守假设），Apple 的盈利质量仍在 97% 以上，不会跌破 0.8 的红旗线。这说明 Apple 的盈利质量在不同假设下都相对稳健。

> **实战建议**：当你分析一家公司时，一定要做这个敏感度分析。如果维持性 CapEx 比例的小幅变化就会让 EQ 从 1.1 跌到 0.7，说明这个估值结论很脆弱，需要更谨慎。

---

## 9. 零增长 vs 含增长估值

valueinvest 库的 Owner Earnings 估值使用两种模型取平均：

**零增长模型**（最保守）：

$$V_0 = \frac{\text{OE per share}}{\text{WACC}}$$

假设公司永远不再增长，把所有 Owner Earnings 作为"永续年金"来估值。这是最保守的底线估计。

**含增长模型**（Gordon Growth Model 变体）：

$$V_g = \frac{\text{OE per share} \times (1 + g)}{\text{WACC} - g}$$

假设公司以增长率 g 永续增长。增长率越高，估值越大，但结果也越不确定。

**最终 Fair Value** = 两种模型的平均值。

![估值模型对比](https://raw.githubusercontent.com/wangzhe3224/valueinvest-learn/main/learn/owner_earnings/plots/growth_valuation.png)

**图表解读**：

- **绿色虚线（零增长）**：\$77.00 — 如果 Apple 不再增长，每股值 \$77
- **蓝色实线（含增长）**：随着增长率假设增加，估值急剧上升
- **红色虚线（当前股价）**：\$246.63 — 市场定价隐含了约 5%+ 的永续增长假设
- **紫色虚线（Fair Value）**：\$96.25 — 零增长和含增长的中间值

> **重要洞察**：当 WACC（10%）接近增长率（15.7%）时，含增长模型的估值会趋向无穷大。对于增长率超过 WACC 的公司，valueinvest 使用了安全上限（零增长价值 × 1.5），避免得出不现实的估值。

---

## 10. 与其他估值方法交叉验证

单一方法总有局限性。让我们看看 Owner Earnings 在所有估值方法中的位置：

```python
results_all = engine.run_all(stock)
for r in results_all:
    if r.fair_value and r.fair_value > 0:
        print(f"{r.method:20s}: \${r.fair_value:>8.2f}  ({r.premium_discount:+.1f}%)  [{r.confidence}]")
```

| 方法 | 内在价值 | 溢价/折价 | 置信度 |
|------|---------|----------|--------|
| Graham Formula | \$262.02 | +6.2% | High |
| DCF (10-Year) | \$103.61 | -58.0% | High |
| Reverse DCF | \$246.63 | 0.0% | High |
| EPV (Zero Growth) | \$74.91 | -69.6% | Low |
| DDM (Gordon Growth) | \$37.51 | -84.8% | High |
| PEG Ratio | \$124.19 | -49.6% | High |
| P/B Valuation | \$112.48 | -54.4% | High |
| **Owner Earnings** | **\$96.25** | **-61.0%** | **Medium** |
| EV/EBITDA | \$125.87 | -49.0% | High |
| PE Relative | \$232.40 | -5.8% | Low |
| PB Relative | \$274.98 | +11.5% | Low |
| **平均** | **\$198.86** | | |

### Owner Earnings 在综合分析中的角色

1. **提供盈利质量视角**：其他方法（如 DCF、Graham）关注估值高低，但 Owner Earnings 关注的是"利润到底有多少是真金白银"
2. **作为"现金现实"的校准基准**：当 Owner Earnings 估值远低于其他方法时，说明市场可能过于乐观
3. **帮助识别利润虚高的公司**：如果 Earnings Quality 很低，其他基于净利润的估值方法（如 P/E、PEG）都会高估公司价值

---

## 11. Owner Earnings 的局限性和最佳实践

### 局限性

| 局限 | 原因 | 影响 |
|------|------|------|
| **维持性 CapEx 难以精确估算** | 公司财报只报告总 CapEx，不区分维持性和增长性 | 这是最大的不确定性来源 |
| **营运资金变动估计简化** | 需要历史数据才能准确计算 ΔNWC | 当前使用 10% NWC 作为代理，可能不够准确 |
| **不适用于高增长公司** | 大量增长性 CapEx 会被误算为维持性 | 可能严重低估高速成长的公司（如早期的 Amazon） |
| **增长率敏感性高** | WACC 接近增长率时估值趋向无穷大 | 含增长模型的结果高度依赖增长率假设 |

### 最佳实践

1. **优先用于成熟公司** — 现金流稳定、CapEx 可预测的公司最适合此方法（如消费品、公用事业）
2. **作为盈利质量过滤器** — Earnings Quality < 0.8 是红旗信号，配合 Piotroski F-Score 使用效果更好
3. **保守估计维持性 CapEx** — 宁可高估维持性 CapEx（降低 Owner Earnings），也不要低估。巴菲特本人倾向于直接用总 CapEx 代替维持性 CapEx
4. **结合其他估值方法** — Owner Earnings + DCF 交叉验证，如果两者结论一致，置信度更高

---

## 12. 总结

### 关键要点

1. **Owner Earnings = Net Income + D&A - Maintenance CapEx - ΔNWC**
   - 衡量公司真正能分配给股东的现金
   - 比净利润更接近经济现实

2. **盈利质量是关键指标**
   - OE/NI > 1.2 = Cash-rich（优秀）
   - OE/NI 0.8-1.2 = Normal（正常）
   - OE/NI < 0.8 = Accrual-heavy（警惕）

3. **维持性 CapEx 的估计是最大的不确定性**
   - 保守估计比精确估计更重要
   - 做敏感度分析检验结果稳健性

4. **适用场景**
   - 成熟、现金流稳定的公司
   - 作为盈利质量过滤器和估值交叉验证

### 快速上手

```python
from valueinvest import Stock, ValuationEngine

engine = ValuationEngine()
stock = Stock.from_api("AAPL")  # 替换为你感兴趣的股票
result = engine.run_single(stock, "owner_earnings")

print(f"Owner Earnings: \${result.details['owner_earnings']/1e9:.2f}B")
print(f"Earnings Quality: {result.details['earnings_quality']:.1%}")
print(f"Fair Value: \${result.fair_value:.2f}")
print(f"Assessment: {result.assessment}")
```

---

**参考资料**：
- [ValueInvest GitHub](https://github.com/wangzhe3224/valueinvest)
- Berkshire Hathaway 1986 Annual Report - Warren Buffett
- 《巴菲特致股东的信》- Warren Buffett
