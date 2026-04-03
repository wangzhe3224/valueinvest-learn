# Graham Number（格雷厄姆数）估值教程

本教程将带你深入了解 **Graham Number** 估值方法——本杰明·格雷厄姆为防御型投资者设计的经典公式。

> 本教程使用 [ValueInvest](https://github.com/wangzhe3224/valueinvest) 库

## 学习目标

1. 理解 Graham Number 的核心原理和推导过程
2. 掌握公式的适用条件和局限性
3. 使用 `valueinvest` 库获取真实数据并估值
4. 手动计算每一步，理解公式背后的逻辑
5. 对比不同公司的 Graham Number 结果
6. 理解资产轻公司为何不适用此方法

---

## 1. Graham Number 基础概念

### 什么是 Graham Number？

Graham Number 是本杰明·格雷厄姆在《聪明的投资者》中提出的估值指标，专门为**防御型投资者**设计。

核心思想非常朴素：一家公司的合理股价应该同时满足**市盈率（P/E）**和**市净率（P/B）**都不过高。

格雷厄姆的智慧在于——他不希望你仅仅因为"市盈率看起来合理"就买入一只股票。很多公司的 P/E 确实不高，但如果它的账面价值很薄（即 P/B 很高），那低 P/E 可能只是盈利暂时虚高造成的假象。反之，P/B 低的股票如果盈利质量差，同样不值得买。**两个维度同时把关**，才能有效过滤掉大部分陷阱。

### 格雷厄姆的防御型投资标准

格雷厄姆认为防御型投资者应该满足：

- **P/E ≤ 15**（市盈率不超过 15 倍）
- **P/B ≤ 1.5**（市净率不超过 1.5 倍）
- **P/E × P/B ≤ 22.5**（两者乘积不超过 22.5）

> 为什么是 15 和 1.5？这是格雷厄姆经过数十年投资实践得出的经验值。他认为，一个防御型投资者为一家公司支付的价格，不应超过其每股收益的 15 倍，同时也不应超过其每股净资产的 1.5 倍。这两个条件共同确保了你在"盈利能力"和"资产安全"两个维度上都不会过度支付。

### 核心公式推导

从 P/E × P/B ≤ 22.5 出发：

$$\frac{P}{E} \times \frac{P}{B} \leq 22.5$$

展开得：

$$\frac{P^2}{E \times B} \leq 22.5$$

因此最大合理价格：

$$P \leq \sqrt{22.5 \times EPS \times BVPS}$$

其中：
- $EPS$ = 每股收益（Earnings Per Share）
- $BVPS$ = 每股净资产（Book Value Per Share）
- 22.5 = 15 × 1.5（P/E 上限 × P/B 上限）

**直觉理解**：这个公式本质上是在说——"我愿意为你的盈利能力付最多 15 倍，同时为你的净资产付最多 1.5 倍，这两个约束的交集决定了你的最高合理股价。"由于两者相乘等于 22.5，开方后恰好对应这两个约束同时满足的边界点。

### 保守区间

| 参数组合 | 乘积 | 含义 |
|----------|------|------|
| P/E=14.1, P/B=1.0 | 14.1 | 非常保守 |
| P/E=14.9, P/B=1.35 | 20.0 | 保守 |
| **P/E=15, P/B=1.5** | **22.5** | **标准（Graham Number）** |
| P/E=16.7, P/B=1.5 | 25.0 | 宽松 |

> 实战建议：如果你是比较谨慎的投资者，可以同时看乘数 20 和 22.5 两个结果。如果当前股价低于乘数 20 算出的 Graham Number，说明安全边际更充足。

---

## 2. 获取真实股票数据

Graham Number 只需要 3 个基本数据：
- **EPS**（每股收益）
- **BVPS**（每股净资产）
- **当前股价**

我们使用 `valueinvest` 库自动获取。选择 JPMorgan Chase (JPM) 作为案例，原因是：银行股资产重、BVPS 高、盈利稳定——正是 Graham Number 的理想标的。

```python
from valueinvest import Stock, ValuationEngine

ticker = "JPM"
stock = Stock.from_api(ticker)

print(f"公司：{stock.name} ({stock.ticker})")
print(f"当前股价：\${stock.current_price:.2f}")
print(f"每股收益 (EPS)：\${stock.eps:.2f}")
print(f"每股净资产 (BVPS)：\${stock.bvps:.2f}")
```

运行结果：

```
公司：JPMorgan Chase & Co. (JPM)
当前股价：\$282.84
每股收益 (EPS)：\$20.03
每股净资产 (BVPS)：\$126.99
市盈率 (P/E)：14.1x
市净率 (P/B)：2.2x
P/E x P/B：31.5
```

> 从初步数据就能看出，JPM 的 P/E 虽然 14.1x 满足 ≤15 的条件，但 P/B 高达 2.2x，远超 1.5 的上限，两者乘积 31.5 也远超 22.5。这意味着按格雷厄姆的标准，JPM 当前可能偏贵。

---

## 3. 理解关键参数

Graham Number 的核心参数是 **22.5** 这个乘数，它来自 P/E 上限和 P/B 上限的乘积。让我们深入理解这些参数。

### 核心乘数 22.5 的含义

22.5 = 15 × 1.5，这不是一个随意的数字，而是两个独立约束的交集：

- **15（P/E 上限）**：格雷厄姆时代，股票市场平均 P/E 约 15 倍。他要求防御型投资者不支付高于平均水平的估值。
- **1.5（P/B 上限）**：为净资产支付 50% 的溢价，是格雷厄姆认为的合理上限。

### BVPS 适用阈值

当 BVPS < \$10 时，公式不适用。原因是：资产轻公司（如科技股大量回购）BVPS 被人为压低，此时 Graham Number 会给出一个不合理的低估结果。

对 JPM 来说：

```
BVPS = \$126.99
>>> 高于阈值，Graham Number 适用

当前快照：
   P/E = 14.1x (格雷厄姆标准: <= 15x) ✓ OK
   P/B = 2.2x (格雷厄姆标准: <= 1.5x) ✗ 偏高
   P/E x P/B = 31.5 (格雷厄姆标准: <= 22.5) ✗ 偏高
```

> 关键洞察：P/E 单独看是"合格"的，但结合 P/B 后就暴露了问题。这正是 Graham Number 的价值所在——**双维度约束比单维度更可靠**。

---

## 4. 运行 Graham Number 估值

使用 `valueinvest` 库的 `ValuationEngine` 一行代码即可完成估值：

```python
from valueinvest import Stock, ValuationEngine

engine = ValuationEngine()
stock = Stock.from_api("JPM")
result = engine.run_single(stock, "graham_number")

print(f"Graham Number：\${result.fair_value:.2f}")
print(f"当前股价：\${result.current_price:.2f}")
print(f"溢价/折价：{result.premium_discount:+.1f}%")
print(f"评估：{result.assessment}")
```

运行结果：

```
============================================================
Graham Number 估值结果：JPMorgan Chase & Co. (JPM)
============================================================

估值结果
   Graham Number：\$239.23
   当前股价：\$282.84
   溢价/折价：-15.4%
   评估：Overvalued

估值区间
   保守 (乘数20)：\$225.55
   基准 (乘数22.5)：\$239.23
   宽松 (乘数25)：\$252.17

详细信息
   公式：√(22.5 × EPS × BVPS)
   EPS：\$20.03
   BVPS：\$126.99
   当前 P/E x P/B：31.5

分析要点
   - Graham's formula for defensive investors (P/E × P/B ≤ 22.5)
   - Conservative range: \$225.55 - \$252.17

置信度：High
适用性：Applicable
```

**结果解读**：
- JPM 的 Graham Number 为 \$239.23，当前股价 \$282.84，**溢价 15.4%**
- 即使在宽松假设下（乘数 25），Graham Number 也只有 \$252.17，仍低于当前股价
- 这意味着按格雷厄姆的防御型标准，JPM 当前定价偏高

---

## 5. 手动计算 Graham Number（理解每一步）

为了让公式不再是一个"黑盒"，让我们一步一步手动计算：

```python
import math

eps = 20.03      # JPM 的每股收益
bvps = 126.99    # JPM 的每股净资产
current_price = 282.84
multiplier = 22.5

# Step 1: 检查前提条件
# EPS > 0 ✓  BVPS > 0 ✓  BVPS > \$10 ✓

# Step 2: 计算乘积
product = multiplier * eps * bvps
# 22.5 × 20.03 × 126.99 = 57,230.32

# Step 3: 开平方
graham_num = math.sqrt(product)
# √57,230.32 = \$239.23

# Step 4: 与当前股价比较
premium = ((graham_num - current_price) / current_price) * 100
# (\$239.23 - \$282.84) / \$282.84 = -15.4%
# 股价高于 Graham Number，可能被高估

# Step 5: 隐含的估值倍数
implied_pe = graham_num / eps    # 11.9x
implied_pb = graham_num / bvps   # 1.9x
# P/E × P/B = 22.5 (= multiplier) ✓
```

详细计算过程：

```
Step 1: 检查前提条件
   EPS = \$20.03 (必须 > 0) ✓
   BVPS = \$126.99 (必须 > 0，建议 > \$10) ✓
   OK: 前提条件满足

Step 2: 计算乘积
   22.5 × 20.03 × 126.99 = 57,230.32

Step 3: 开平方
   √57,230.32 = \$239.23

Step 4: 与当前股价比较
   Graham Number: \$239.23
   当前股价: \$282.84
   溢价/折价: -15.4%
   >>> 股价高于 Graham Number，可能被高估

Step 5: Graham Number 隐含的估值倍数
   隐含 P/E = \$239.23 / \$20.03 = 11.9x
   隐含 P/B = \$239.23 / \$126.99 = 1.9x
   P/E × P/B = 22.5 (= 22.5)
```

> 注意 Step 5 的隐含倍数：Graham Number 对应的 P/E 只有 11.9x、P/B 只有 1.9x。虽然 P/B 仍略高于 1.5，但这是因为格雷厄姆要求的是"乘积不超过 22.5"，而不是两个维度分别达标。在乘积约束下，P/E 低一些可以补偿 P/B 高一些。

---

## 6. 可视化 Graham Number

![Graham Number Valuation](https://raw.githubusercontent.com/wangzhe3224/valueinvest-learn/main/learn/graham_number/plots/graham_number_valuation.png)

**左图解读**：
- 绿色柱（标准 22.5）为 Graham Number \$239.23
- 红色柱（当前股价）为 \$282.84
- 即使在最宽松假设下（乘数 25），Graham Number 也只有 \$252.17，仍低于当前价格
- 这直观地展示了 JPM 当前股价的溢价程度

**右图解读**：
- 绿色曲线是格雷厄姆的"安全边界"：P/E × P/B = 22.5
- 绿色阴影区域是"安全区域"——任何位于这条曲线下方的股票都满足格雷厄姆的防御型标准
- 红色点（JPM 当前位置）位于边界线的**上方**，即 P/E × P/B = 31.5，超出了安全区域
- 绿色星号是 Graham Number 对应的估值点，恰好在边界线上

---

## 7. 多公司对比分析

Graham Number 的一个优势是**简单可比**。让我们对比四只主要金融股：

```python
tickers = ["JPM", "BAC", "WFC", "GS"]

for t in tickers:
    s = Stock.from_api(t)
    gn = math.sqrt(22.5 * s.eps * s.bvps)
    premium = ((gn - s.current_price) / s.current_price) * 100
    print(f"{t}: Price=\${s.current_price:.2f}, GN=\${gn:.2f}, Premium={premium:+.1f}%")
```

| Ticker | 公司 | 股价 | EPS | BVPS | P/E | P/B | Graham# | 溢价% |
|--------|------|------|-----|------|-----|-----|---------|-------|
| JPM | JPMorgan Chase | \$282.84 | \$20.03 | \$126.99 | 14.1x | 2.2x | \$239.23 | -15.4% |
| BAC | Bank of America | \$46.97 | \$3.81 | \$38.44 | 12.3x | 1.2x | \$57.41 | +22.2% |
| WFC | Wells Fargo | \$77.19 | \$6.26 | \$53.19 | 12.3x | 1.5x | \$86.56 | +12.1% |
| GS | Goldman Sachs | \$802.89 | \$51.30 | \$356.47 | 15.7x | 2.3x | \$641.45 | -20.1% |

![Multi-Company Comparison](https://raw.githubusercontent.com/wangzhe3224/valueinvest-learn/main/learn/graham_number/plots/multi_company_comparison.png)

**对比分析**：

1. **BAC（美国银行）**：Graham Number 最高溢价 +22.2%，是目前四只银行股中最被低估的。P/E 12.3x 和 P/B 1.2x 都在格雷厄姆标准内。

2. **WFC（富国银行）**：Graham Number 溢价 +12.1%，P/B 恰好 1.5x，刚好踩在格雷厄姆标准的边界上。

3. **JPM（摩根大通）**：溢价 -15.4%，P/E 合格但 P/B 偏高（2.2x），市场给予的溢价反映了其卓越的管理层和业务质量。

4. **GS（高盛）**：溢价 -20.1%，P/E 和 P/B 都偏高。高盛的盈利波动性较大，且 ROE 结构与纯银行不同，市场给予的成长性溢价更高。

> 实战提示：Graham Number 更适合在**同行业内**做横向比较。不要拿银行股的 Graham Number 和科技股的做对比——不同行业的 P/E、P/B 中位数本身就不同。

---

## 8. 资产轻公司为什么不适用？

Graham Number 的一个重要局限：**不适用于资产轻公司**。让我们用苹果公司来演示。

```python
from valueinvest import Stock, ValuationEngine

stock_aapl = Stock.from_api("AAPL")
engine = ValuationEngine()
result_aapl = engine.run_single(stock_aapl, "graham_number")

print(f"公司：{stock_aapl.name}")
print(f"当前股价：\${stock_aapl.current_price:.2f}")
print(f"EPS：\${stock_aapl.eps:.2f}")
print(f"BVPS：\${stock_aapl.bvps:.2f}")
print(f"P/E：{stock_aapl.current_price/stock_aapl.eps:.1f}x")
print(f"P/B：{stock_aapl.current_price/stock_aapl.bvps:.1f}x")
```

运行结果：

```
============================================================
案例：Apple Inc. (AAPL)
============================================================
当前股价：\$248.80
EPS：\$7.91
BVPS：\$6.00
P/E：31.5x
P/B：41.5x

--- 分析 ---
BVPS = \$6.00，远低于 \$10 阈值

原因：苹果长期进行大规模股票回购，导致：
  1. 股东权益被消耗（回购冲减留存收益）
  2. BVPS 被人为压低
  3. P/B 比率被人为拔高（目前 41.5x）

--- 运行结果 ---
不适用：N/A - Graham Number not applicable: BVPS (\$6.00) is below
\$10 threshold. This typically indicates an asset-light company (e.g.,
tech with massive buybacks) where the formula is unreliable.
```

**为什么苹果的 BVPS 这么低？**

苹果在过去十年中通过大规模股票回购，累计回购了数千亿美元的股票。回购在会计上会减少股东权益（借：股本/留存收益，贷：现金），导致：

1. **账面价值持续下降**：苹果的 BVPS 从多年前的 \$20+ 降到了现在的 \$6
2. **P/B 失去参考意义**：41.5x 的 P/B 看起来很夸张，但这并不代表苹果的资产被高估——只是账面价值被回购人为压缩了
3. **Graham Number 公式失效**：BVPS 太低导致 Graham Number 算出的结果不合理

**结论：Graham Number 不适用于以下类型的公司**：

| 公司类型 | 原因 | 典型代表 |
|----------|------|----------|
| 大规模回购的科技公司 | BVPS 被回购压低 | AAPL, MSFT, GOOG |
| 无形资产占比高的公司 | 品牌、专利未计入账面价值 | KO, PG, NKE |
| 高增长公司 | 未来价值远大于当前账面价值 | AMZN, NVDA |
| SaaS/软件公司 | 几乎没有有形资产 | CRM, ADBE |

---

## 9. A 股案例

Graham Number 同样适用于 A 股市场。让我们分析兴业银行（601166）：

```python
from valueinvest import Stock, ValuationEngine

stock_cn = Stock.from_api("601166")
engine = ValuationEngine()
result_cn = engine.run_single(stock_cn, "graham_number")

print(f"公司：{stock_cn.name} ({stock_cn.ticker})")
print(f"当前股价：{stock_cn.current_price:.2f}")
print(f"每股收益：{stock_cn.eps:.2f}")
print(f"每股净资产：{stock_cn.bvps:.2f}")
print(f"P/E：{stock_cn.current_price/stock_cn.eps:.1f}x")
print(f"P/B：{stock_cn.current_price/stock_cn.bvps:.1f}x")
```

运行结果：

```
============================================================
公司：兴业银行 (601166)
============================================================
当前股价：18.69
每股收益：3.46
每股净资产：42.61
P/E：5.4x
P/B：0.4x

Graham Number 估值结果
   Graham Number：57.59
   溢价/折价：+208.2%
   评估：Undervalued
   保守：54.30
   宽松：60.71
```

**分析**：

兴业银行的 Graham Number 显示了 **+208.2% 的溢价**（即当前股价 18.69 远低于 Graham Number 57.59），这是一个极端的"低估"信号。

为什么会这样？注意关键数据：
- **P/E = 5.4x**：远低于 15 倍标准
- **P/B = 0.4x**：远低于 1.5 倍标准，甚至低于 1

P/B 低于 1 意味着你在市场上可以用低于账面价值的价格买入这家银行。这在 A 股银行板块中非常普遍，原因是：
1. 市场对银行坏账率的担忧
2. 银行的盈利增长预期较低
3. A 股整体对银行股的估值折价

> 重要提醒：Graham Number 显示"低估"并不等于"应该买入"。A 股银行股长期处于低估值状态有其结构性原因。Graham Number 是一个**参考指标**，买入决策还需要结合宏观经济、行业前景和个股基本面综合判断。

---

## 10. Graham Number 的局限性和最佳实践

### 局限性

**1. 仅适用于资产重型公司**
BVPS < \$10 的公司公式失效。不适合大量回购或无形资产占比高的企业。

**2. 忽略增长**
公式完全不考虑未来增长潜力。对稳定增长的公司可能过于保守，对衰退中的公司可能过于乐观（因为用的是当期 EPS）。

**3. 单一时点快照**
只使用当前 EPS 和 BVPS，不考虑盈利波动或周期性。如果一家公司恰好处于盈利周期高点，算出的 Graham Number 会偏高。

**4. 22.5 乘数的假设**
P/E=15 和 P/B=1.5 的上限是经验值，来自格雷厄姆时代的市场环境。当前市场环境下，优质公司的估值中枢可能已经发生了变化。

### 最佳实践

**1. 选对标的**
优先用于银行、保险、传统制造业。避免用于科技、消费品牌等资产轻公司。

**2. 作为筛选工具**
Graham Number 更适合初筛，而非精确估值。股价低于 Graham Number 不一定值得买，但股价大幅高于 Graham Number 往往值得警惕。

**3. 结合其他方法**
| 组合方式 | 目的 |
|----------|------|
| Graham Number + DCF | 结合保守基准和增长预期 |
| Graham Number + Piotroski F-Score | 检查财务健康度 |
| Graham Number + 分红收益率 | 验证股东回报能力 |

**4. 关注趋势**
观察 EPS 和 BVPS 的历史趋势。盈利下降的公司 Graham Number 会持续走低，形成"价值陷阱"。

---

## 11. 综合估值分析

将 Graham Number 与其他估值方法结合，可以得到更全面的视角。以 JPM 为例：

```python
from valueinvest import Stock, ValuationEngine
import pandas as pd
import numpy as np

engine = ValuationEngine()
stock = Stock.from_api("JPM")
results_all = engine.run_all(stock)

valid_results = [(r.method, r.fair_value, r.premium_discount, r.confidence)
                 for r in results_all if r.fair_value and r.fair_value > 0]

df_results = pd.DataFrame(valid_results,
                          columns=['方法', '内在价值', '溢价/折价%', '置信度'])
print(df_results.to_string(index=False))
```

| 方法 | 内在价值 | 溢价/折价% | 置信度 |
|------|----------|------------|--------|
| Graham Number | \$239.23 | -15.4% | High |
| Graham Formula | \$224.49 | -20.6% | High |
| EPV (Zero Growth) | \$162.98 | -42.4% | Low |
| Two-Stage DDM | \$83.88 | -70.3% | Medium |
| PEG Ratio | \$50.08 | -82.3% | Medium |
| GARP | \$231.46 | -18.2% | Low |
| Rule of 40 | \$282.84 | 0.0% | Medium |
| P/B Valuation | \$224.36 | -20.7% | High |
| Residual Income | \$188.36 | -33.4% | Medium |
| Magic Formula | \$218.78 | -22.6% | Medium |
| Owner Earnings | \$225.17 | -20.4% | Medium |
| EV/EBITDA | \$274.15 | -3.1% | Low |

**综合分析要点**：

- **Graham Number（\$239.23）** 在所有方法中处于中上水平，提供了一个"保守但合理"的价值参考
- **Graham Formula（\$224.49）** 更加保守，因为它额外考虑了增长要求
- **多数方法**给出的估值都在 \$220-240 区间，说明 JPM 当前 \$282 的股价确实存在溢价
- **EV/EBITDA（\$274.15）** 最接近当前股价，反映了 JPM 的盈利质量和现金流强劲

> Graham Number 在综合分析中的角色：提供一个**保守的价值锚点**。当其他方法给出的估值都高于 Graham Number 时，说明市场确实在为某些"超额因素"（如管理质量、品牌价值、增长预期）付费。

---

## 12. 总结

### 关键要点

**1. Graham Number = √(22.5 × EPS × BVPS)**
来自 P/E ≤ 15 且 P/B ≤ 1.5 的约束，为防御型投资者提供保守的价格上限。

**2. 适用条件**
- EPS > 0，BVPS > \$10
- 资产重型、盈利稳定的成熟公司
- 银行、保险、传统制造业

**3. 不适用场景**
- 大规模回购的科技公司（BVPS 被压低）
- 无形资产占比高的企业（账面价值不反映真实价值）
- 高增长公司（未来价值远大于当前账面价值）

**4. 使用建议**
- 作为初筛工具，不作为唯一估值标准
- 结合 DCF、相对估值等其他方法
- 关注 EPS 和 BVPS 的趋势，警惕"价值陷阱"
- 在同行业内做横向比较，而非跨行业

### Graham Number 使用决策树

```
开始
 │
 ├─ BVPS >= \$10？
 │   ├─ 否 → 不适用，换用 DCF 或相对估值
 │   └─ 是 ↓
 │
 ├─ 公司类型？
 │   ├─ 银行/保险/传统制造 → 适用 ✓
 │   ├─ 科技/品牌消费 → 不适用 ✗
 │   └─ 其他 → 谨慎使用
 │
 ├─ 股价 < Graham Number？
 │   ├─ 是 → 可能低估，结合其他方法验证
 │   └─ 否 → 可能高估或合理，看溢价幅度
 │
 └─ 结合其他估值方法做最终判断
```

---

**参考资料**：
- [ValueInvest GitHub](https://github.com/wangzhe3224/valueinvest)
- 《聪明的投资者》- Benjamin Graham
- 《证券分析》- Benjamin Graham & David Dodd
