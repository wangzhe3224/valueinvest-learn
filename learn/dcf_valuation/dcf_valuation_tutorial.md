# DCF（现金流折现）估值模型教程

本教程将带你深入了解 **Discounted Cash Flow (DCF)** 估值方法，并使用真实股票数据进行实践。

## 学习目标

1. 理解 DCF 的核心原理
2. 掌握关键参数的设定
3. 使用 `valueinvest` 库获取真实数据
4. 进行敏感性分析
5. 理解 DCF 的局限性和适用场景

## 为什么学 DCF？

在投资世界中，你每天都会听到这样的对话：

> "这只股票太便宜了，PE 才 10 倍！"
> "这家公司估值 100 倍 PS，泡沫太大了！"

PE、PB、PS 这些**相对估值法**确实简单好用，但它们有一个致命缺陷——它们只能告诉你"相对便宜"还是"相对贵"，就像拿一支温度计去比较两杯水哪个更热，却永远不知道它们到底几度。

**DCF 是那支能告诉你"绝对温度"的温度计。**

它不依赖市场给其他公司的定价，而是直接问一个根本问题：**这家公司未来能赚多少钱，这些钱今天值多少？**

Warren Buffett 说过："评估一家企业的价值，就是评估它整个生命周期内能产生的自由现金流，并以适当的利率折现到现在。" 这就是 DCF 的本质。

当然，DCF 也有明显的局限性——"Garbage in, garbage out"。你对未来的预测有多准，DCF 的结果就有多准。这也是为什么本教程会花大量篇幅讲解参数设定和敏感性分析。

---

> 本教程使用 [ValueInvest](https://github.com/wangzhe3224/valueinvest) 库，所有代码和数据均可复现。

## 1. DCF 基础概念

### 什么是 DCF？

DCF（Discounted Cash Flow，现金流折现）是一种**绝对估值方法**，核心思想是：

**一家公司的价值 = 未来所有现金流的现值之和**

### 为什么要"折现"？

这是理解 DCF 的关键门槛。一个简单的例子：

假设一家公司承诺 1 年后给你 **110 元**，年利率是 10%。这 110 元今天值多少钱？

$$\text{现值} = \frac{110}{1 + 10\%} = 100 \text{ 元}$$

也就是说，1 年后的 110 元和今天的 100 元是等价的。这就是**货币的时间价值**——同样的钱，越早拿到越值钱，因为你可以拿去投资赚取回报。

DCF 的本质就是对一家公司未来每年产生的现金流，逐一"折现"到今天的价值，然后加总。**折现率（r）越高，未来现金流"缩水"越多，公司估值就越低。** 这也符合直觉：风险越大的公司，你要求的回报越高，它在你眼里就越不值钱。

### 核心公式

$$\text{企业价值} = \sum_{t=1}^{n} \frac{FCF_t}{(1+r)^t} + \frac{TV}{(1+r)^n}$$

其中：
- $FCF_t$ = 第 t 年的自由现金流
- $r$ = 折现率（WACC）
- $n$ = 预测期（通常 10 年）
- $TV$ = 终值（Terminal Value）

公式看起来只有一行，但实际拆成两大部分：

| 部分 | 含义 | 说明 |
|------|------|------|
| $\sum_{t=1}^{n} \frac{FCF_t}{(1+r)^t}$ | **预测期现值** | 未来 10 年每年现金流的折现值之和 |
| $\frac{TV}{(1+r)^n}$ | **终值现值** | 第 10 年之后所有现金流的折现值 |

### 终值计算

使用 **Gordon Growth Model**：

$$TV = \frac{FCF_n \times (1 + g)}{r - g}$$

- $g$ = 永续增长率（通常 2-3%）

你可能会问：**为什么只预测 10 年？10 年之后呢？**

答案是：没有人能准确预测 10 年后的事情。但公司通常不会在第 10 年突然消失——它会继续运营。所以我们用一个"终值"来概括第 11 年到无穷远的所有现金流。

终值计算假设公司在第 10 年后进入"稳定增长"阶段，每年以一个固定的永续增长率 $g$ 增长。**注意公式中的分母 $(r - g)$：折现率必须大于永续增长率，否则这个公式会趋向无穷大，这在数学上和经济学上都没有意义。** 换句话说，没有任何公司能永远以高于其资本成本的速度增长。

### 从企业价值到每股价值

DCF 算出的是**企业价值（Enterprise Value）**，但你真正关心的是**每股价值**。转换过程很简单：

$$\text{每股价值} = \frac{\text{企业价值} - \text{净债务}}{\text{总股数}}$$

为什么要减去净债务？因为你买股票买的是公司的"股权"，而企业价值包含了债权人的部分。减去净债务（总债务 - 现金）后，剩下的才是属于股东的。

```python
# 导入必要的库
from valueinvest import Stock, ValuationEngine
from valueinvest.valuation.dcf import DCF, ReverseDCF
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# 设置中文显示
plt.rcParams['font.sans-serif'] = ['Arial Unicode MS', 'SimHei', 'DejaVu Sans']
plt.rcParams['axes.unicode_minus'] = False

print("✓ 库加载成功")
```

## 2. 获取真实股票数据

我们使用 `valueinvest` 库获取真实股票数据。支持：
- **A股**：通过 AKShare（免费）
- **美股**：通过 yfinance（免费）

```python
# 获取苹果公司 (AAPL) 数据
ticker = "AAPL"
print(f"正在获取 {ticker} 数据...\n")

stock = Stock.from_api(ticker)

# 显示基本信息
print("=" * 60)
print(f"公司：{stock.name} ({stock.ticker})")
print("=" * 60)
print(f"当前股价：${stock.current_price:.2f}")
print(f"总股数：{stock.shares_outstanding/1e9:.2f}B")
print(f"市值：${stock.market_cap/1e9:.2f}B")
print(f"\n财务数据：")
print(f"  自由现金流 (FCF)：${stock.fcf/1e9:.2f}B")
print(f"  净债务：${stock.net_debt/1e9:.2f}B")
print(f"  每股收益 (EPS)：${stock.eps:.2f}")
print(f"  每股净资产 (BVPS)：${stock.bvps:.2f}")
```

输出：

```
正在获取 AAPL 数据...

============================================================
公司：Apple Inc. (AAPL)
============================================================
当前股价：\$248.80
总股数：14.68B
市值：\$3652.67B

财务数据：
  自由现金流 (FCF)：\$106.31B
  净债务：\$62.72B
  每股收益 (EPS)：\$7.91
  每股净资产 (BVPS)：\$6.00
```

**解读这些数据：**

- **自由现金流 \$106.31B**：这是苹果过去一年真正"赚到口袋里"的现金，扣除了所有运营开支和资本支出。它是 DCF 模型的核心输入——注意，不是净利润（Net Income），而是自由现金流。为什么？因为利润是会计概念，可以有很多"水分"（比如大量的应收账款），而自由现金流是真金白银。
- **净债务 \$62.72B**：苹果虽然账上有大量现金，但也有债务。净债务 = 总债务 - 现金及等价物。正值表示净负债，需要从企业价值中扣除才能得到股权价值。
- **EPS \$7.91 vs BVPS \$6.00**：EPS 远高于 BVPS，这在轻资产/高 ROE 的科技公司很常见，意味着苹果靠少量资产就能创造大量利润。

## 3. 理解 DCF 的关键参数

DCF 模型需要设定 4 个关键参数，这些参数会显著影响估值结果。**可以说，设定参数的过程本身就是投资分析的过程——你对参数的选择，反映了你对这家公司的理解。**

```python
# DCF 关键参数设定

print("📊 DCF 关键参数")
print("=" * 60)

# 1. 增长率 (Years 1-5)
# 通常是公司历史增长率和行业预期的结合
# 苹果是成熟公司，假设未来5年年均增长 8%
growth_rate_1_5 = 8.0  # %
print(f"1. 增长率 (Years 1-5): {growth_rate_1_5}%")
print(f"   - 反映公司未来5年的预期增长")
print(f"   - 需要参考历史增长、行业趋势、公司战略")

# 2. 增长率 (Years 6-10)
# 随着公司成熟，增长率通常会放缓
growth_rate_6_10 = 4.0  # %
print(f"\n2. 增长率 (Years 6-10): {growth_rate_6_10}%")
print(f"   - 后期增长通常会放缓")
print(f"   - 一般设为前期增长率的一半左右")

# 3. 终值增长率 (Terminal Growth Rate)
# 永续增长率，通常等于长期通胀率或GDP增长率
terminal_growth = 2.5  # %
print(f"\n3. 终值增长率: {terminal_growth}%")
print(f"   - 代表公司进入稳定期后的永续增长")
print(f"   - 通常设定为 2-3%（接近长期通胀/GDP增长）")

# 4. 折现率 (Discount Rate / WACC)
# 资金的机会成本，通常使用 WACC 或要求的回报率
discount_rate = 10.0  # %
print(f"\n4. 折现率 (WACC): {discount_rate}%")
print(f"   - 代表投资者的机会成本")
print(f"   - 可以用 CAPM 计算，或简单用 8-12%")

print("\n" + "=" * 60)
print("⚠️  参数设定提示：")
print("   - 这些参数会极大影响估值结果")
print("   - 应该进行敏感性分析（稍后演示）")
print("   - 折现率必须大于终值增长率，否则模型无意义")
```

输出：

```
📊 DCF 关键参数
============================================================
1. 增长率 (Years 1-5): 8.0%
   - 反映公司未来5年的预期增长
   - 需要参考历史增长、行业趋势、公司战略

2. 增长率 (Years 6-10): 4.0%
   - 后期增长通常会放缓
   - 一般设为前期增长率的一半左右

3. 终值增长率: 2.5%
   - 代表公司进入稳定期后的永续增长
   - 通常设定为 2-3%（接近长期通胀/GDP增长）

4. 折现率 (WACC): 10.0%
   - 代表投资者的机会成本
   - 可以用 CAPM 计算，或简单用 8-12%

============================================================
⚠️  参数设定提示：
   - 这些参数会极大影响估值结果
   - 应该进行敏感性分析（稍后演示）
   - 折现率必须大于终值增长率，否则模型无意义
```

### 四个参数怎么定？实战经验

**参数 1：前 5 年增长率（最重要的参数）**

不要拍脑袋定。可以从以下几个维度交叉验证：

- **历史增长率**：公司过去 3-5 年的营收/FCF 复合增长率是多少？
- **行业预期**：分析师 consensus 是多少？（可以参考 Yahoo Finance、Wind 等平台）
- **公司指引**：管理层自己怎么说的？（看 earnings call transcript）
- **TAM 空间**：公司所在的市场还有多大增长空间？

一个实用原则：**对于成熟公司，前 5 年增长率通常设为历史增长率打 7-8 折。** 因为树不会长到天上去。比如苹果过去几年 FCF 增长大约 10-12%，我们保守设为 8%。

**参数 2：后 5 年增长率**

一个经验法则：**后 5 年增长率通常设为前 5 年的一半左右。** 随着公司体量变大，维持高增长越来越难。微软在 2010 年代后期增长显著放缓，苹果在 iPhone 高峰期之后也面临类似挑战。

**参数 3：永续增长率（Terminal Growth Rate）**

这是最容易犯错的参数。记住：**这个数字代表的是"到天荒地老"的增长率。** 没有任何公司能永远以 5% 以上的速度增长。

- 美国：长期 GDP 增长约 2-3%，所以 2.0-2.5% 是最常用的设定
- 中国：考虑到更高的名义 GDP 增长，可以用 2.0-3.0%
- **切记：永续增长率不能超过折现率**，否则公式发散

**参数 4：折现率（WACC）**

折现率代表你的**机会成本**——如果你不买这只股票，你在其他同等风险的投资上能赚多少？

简化版的 CAPM 公式：$r = 无风险利率 + \beta \times 市场风险溢价$

- 无风险利率：10 年期国债收益率（美国约 4-4.5%，中国约 2-2.5%）
- 市场风险溢价：历史平均约 5-7%
- $\beta$：衡量股票相对大盘的波动（>1 表示比大盘更波动）

对于不想算 CAPM 的投资者，一个简单的替代方案：

| 公司类型 | 建议折现率 |
|---------|-----------|
| 大型稳定蓝筹（可口可乐、伊利） | 8-9% |
| 成熟科技公司（苹果、微软） | 9-10% |
| 中型成长公司 | 10-12% |
| 小型/高波动公司 | 12-15% |

## 4. 运行 DCF 估值

现在让我们使用这些参数进行 DCF 估值：

```python
# 设置估值参数
stock.growth_rate_1_5 = growth_rate_1_5
stock.growth_rate_6_10 = growth_rate_6_10
stock.terminal_growth = terminal_growth
stock.discount_rate = discount_rate

# 方法1：使用 ValuationEngine
engine = ValuationEngine()
result = engine.run_single(stock, "dcf")

# 显示结果
print("=" * 60)
print(f"DCF 估值结果：{stock.name} ({stock.ticker})")
print("=" * 60)

if result.fair_value:
    print(f"\n💰 估值结果")
    print(f"   内在价值：${result.fair_value:.2f}")
    print(f"   当前股价：${result.current_price:.2f}")
    print(f"   溢价/折价：{result.premium_discount:+.1f}%")
    print(f"   评估：{result.assessment}")

    if result.fair_value_range:
        print(f"\n📊 估值区间")
        print(f"   保守：${result.fair_value_range.low:.2f}")
        print(f"   基准：${result.fair_value_range.base:.2f}")
        print(f"   乐观：${result.fair_value_range.high:.2f}")

    print(f"\n📈 详细信息")
    print(f"   FCF 现值：${result.components['pv_fcf']/1e9:.2f}B")
    print(f"   终值现值：${result.components['pv_terminal']/1e9:.2f}B")
    print(f"   企业价值：${result.components['enterprise_value']/1e9:.2f}B")
    print(f"   股权价值：${result.components['equity_value']/1e9:.2f}B")
    print(f"   终值占比：{result.details['terminal_value_pct']:.1f}%")

    print(f"\n📝 分析要点")
    for analysis in result.analysis:
        print(f"   • {analysis}")

    print(f"\n🎯 置信度：{result.confidence}")
else:
    print(f"❌ 估值失败：{result.error}")
```

输出：

```
============================================================
DCF 估值结果：Apple Inc. (AAPL)
============================================================

💰 估值结果
   内在价值：\$126.22
   当前股价：\$248.80
   溢价/折价：-49.3%
   评估：Overvalued

📊 估值区间
   保守：\$84.94
   基准：\$126.22
   乐观：\$212.52

📈 详细信息
   FCF 现值：\$914.41B
   终值现值：\$1001.40B
   企业价值：\$1915.81B
   股权价值：\$1853.09B
   终值占比：52.3%

📝 分析要点
   • 10-year DCF with terminal value
   • Terminal Value represents 52.3% of total value
   • FCF Year 10: 190.05B

🎯 置信度：Medium
```

### 解读结果

DCF 给出苹果的内在价值为 **\$126.22**，而当前股价是 **\$248.80**，折价 -49.3%。这意味着按照我们的假设，苹果当前股价几乎是"合理价值"的两倍。

**但先别急着下结论！** 这个结果说明了 DCF 的几个重要特点：

1. **估值区间比单一数字更重要**：乐观假设下（\$212.52）离当前股价差距不大，说明如果市场对苹果的增长预期比我们乐观，当前的定价并非完全不合理。

2. **终值占比 52.3%**：超过一半的企业价值来自第 10 年之后的现金流。这意味着估值高度依赖"永续增长"这个假设。下一节的敏感性分析会进一步揭示这一点。

3. **"Overvalued" 不等于"不能买"**：DCF 只是一个参考框架。市场可能看到了你没看到的东西（比如 Apple Intelligence 带来的新增长曲线），也可能纯粹是情绪推动。DCF 的价值在于帮你建立一个"锚点"。

## 5. 手动计算 DCF（理解每一步）

为了深入理解 DCF，让我们手动计算每一步。**强烈建议你跟着算一遍——只有亲手算过，才能真正理解每个数字的含义。**

```python
# 手动 DCF 计算

def manual_dcf(fcf, shares, net_debt, g1, g2, g_term, r):
    """
    手动计算 DCF

    参数:
    - fcf: 当前自由现金流
    - shares: 总股数
    - net_debt: 净债务
    - g1: 前5年增长率 (小数)
    - g2: 后5年增长率 (小数)
    - g_term: 终值增长率 (小数)
    - r: 折现率 (小数)
    """

    # 1. 计算未来10年的 FCF
    projected_fcf = fcf
    cash_flows = []

    for year in range(1, 11):
        if year <= 5:
            projected_fcf *= (1 + g1)
        else:
            projected_fcf *= (1 + g2)

        # 折现到现值
        pv = projected_fcf / ((1 + r) ** year)
        cash_flows.append({
            'year': year,
            'fcf': projected_fcf,
            'pv': pv
        })

    # 2. 计算 FCF 现值总和
    total_pv_fcf = sum(cf['pv'] for cf in cash_flows)

    # 3. 计算终值
    fcf_year_10 = cash_flows[-1]['fcf']
    terminal_value = (fcf_year_10 * (1 + g_term)) / (r - g_term)
    pv_terminal = terminal_value / ((1 + r) ** 10)

    # 4. 计算企业价值
    enterprise_value = total_pv_fcf + pv_terminal

    # 5. 计算股权价值
    equity_value = enterprise_value - net_debt

    # 6. 计算每股内在价值
    intrinsic_value = equity_value / shares

    return {
        'cash_flows': cash_flows,
        'total_pv_fcf': total_pv_fcf,
        'terminal_value': terminal_value,
        'pv_terminal': pv_terminal,
        'enterprise_value': enterprise_value,
        'equity_value': equity_value,
        'intrinsic_value': intrinsic_value,
        'terminal_pct': pv_terminal / enterprise_value * 100
    }

# 执行计算
result_manual = manual_dcf(
    fcf=stock.fcf,
    shares=stock.shares_outstanding,
    net_debt=stock.net_debt,
    g1=growth_rate_1_5/100,
    g2=growth_rate_6_10/100,
    g_term=terminal_growth/100,
    r=discount_rate/100
)

# 创建现金流表格
df_flows = pd.DataFrame(result_manual['cash_flows'])
df_flows['fcf_billion'] = df_flows['fcf'] / 1e9
df_flows['pv_billion'] = df_flows['pv'] / 1e9

print("\n📊 未来10年现金流预测")
print("=" * 60)
print(df_flows[['year', 'fcf_billion', 'pv_billion']].to_string(index=False, float_format='%.2f'))

print(f"\n💰 估值构成")
print("=" * 60)
print(f"FCF 现值总和：${result_manual['total_pv_fcf']/1e9:.2f}B")
print(f"终值：${result_manual['terminal_value']/1e9:.2f}B")
print(f"终值现值：${result_manual['pv_terminal']/1e9:.2f}B")
print(f"企业价值：${result_manual['enterprise_value']/1e9:.2f}B")
print(f"股权价值：${result_manual['equity_value']/1e9:.2f}B")
print(f"\n每股内在价值：${result_manual['intrinsic_value']:.2f}")
print(f"终值占比：{result_manual['terminal_pct']:.1f}%")
```

输出：

```
📊 未来10年现金流预测
============================================================
 year  fcf_billion  pv_billion
    1       114.82      104.38
    2       124.00      102.48
    3       133.92      100.62
    4       144.64       98.79
    5       156.21       96.99
    6       162.46       91.70
    7       168.95       86.70
    8       175.71       81.97
    9       182.74       77.50
   10       190.05       73.27

💰 估值构成
============================================================
FCF 现值总和：\$914.41B
终值：\$2597.37B
终值现值：\$1001.40B
企业价值：\$1915.81B
股权价值：\$1853.09B

每股内在价值：\$126.22
终值占比：52.3%
```

### 逐行解读现金流表

这张表是 DCF 的灵魂，值得仔细看：

| 年份 | 预测 FCF | 现值 (PV) | 观察 |
|------|---------|----------|------|
| 1 | \$114.82B | \$104.38B | 第 1 年的 FCF 增长了 8%，折现后缩水约 9% |
| 5 | \$156.21B | \$96.99B | 注意：第 5 年的 FCF 远大于第 1 年，但现值反而更小！ |
| 6 | \$162.46B | \$91.70B | 增长率从 8% 降到 4%，现值跳降 |
| 10 | \$190.05B | \$73.27B | 第 10 年近 \$1900 亿的现金流，折现后只剩 \$730 亿 |

**核心观察**：FCF 虽然每年都在增长（从 \$114.82B 到 \$190.05B），但现值（PV）却在持续下降。这就是折现的力量——**越远的钱越不值钱**。第 10 年的现金流虽然金额最大，但折现到今天只值第 1 年的 70%。

这也解释了为什么**终值占比高达 52.3%**：未来 10 年我们只能"看清"约一半的价值，另一半藏在第 10 年之后的模糊地带。

### 估值计算链条

```
FCF 现值 (\$914.41B) + 终值现值 (\$1001.40B) = 企业价值 (\$1915.81B)
企业价值 (\$1915.81B) - 净债务 (\$62.72B) = 股权价值 (\$1853.09B)
股权价值 (\$1853.09B) ÷ 总股数 (14.68B) = 每股内在价值 (\$126.22)
```

每一步都很直观，但每一步都藏着一个假设。改变任何一个假设，最终结果都会大不相同。

## 6. 可视化现金流预测

![现金流预测与企业价值构成](https://raw.githubusercontent.com/wangzhe3224/valueinvest-learn/main/learn/dcf_valuation/plots/cashflow_visualization.png)

**左图解读**：蓝色柱子（预测 FCF）逐年增长，而红色柱子（现值 PV）却逐年缩短。这个"剪刀差"直观展示了折现的效果——时间越远，"缩水"越严重。

**右图解读**：企业价值中，FCF 现值和终值现值几乎各占一半。终值占比 52.3% 属于中等水平（一般 DCF 终值占比在 50-70%）。如果终值占比超过 70%，说明你的估值几乎完全依赖于"这家公司会永远活下去并且增长"这个假设，此时需要格外谨慎。

```python
# 关键洞察
print(f"\n💡 关键洞察：")
print(f"   终值占企业价值的 {result_manual['terminal_pct']:.1f}%")
if result_manual['terminal_pct'] > 60:
    print(f"   ⚠️  终值占比过高，估值对终值增长率假设非常敏感")
else:
    print(f"   ✓ 终值占比合理，估值相对稳健")
```

## 7. 敏感性分析

> **如果 DCF 只学一节，就学这一节。**

DCF 对关键参数非常敏感。不同的合理假设可以导致估值结果天差地别。**敏感性分析不是锦上添花，而是 DCF 的标配。** 没有做过敏感性分析的 DCF，就像没有误差范围的测量值——看起来精确，实际上毫无意义。

让我们分析不同参数假设下的估值范围：

```python
# 敏感性分析

# 定义参数范围
growth_rates = [5, 8, 11, 14]  # 前5年增长率
discount_rates = [8, 10, 12]   # 折现率

# 创建敏感性矩阵
sensitivity_matrix = []

for g in growth_rates:
    row = []
    for r in discount_rates:
        # 使用 DCF 类计算
        dcf = DCF(growth_1_5=g, growth_6_10=g*0.5,
                  terminal_growth=terminal_growth, discount_rate=r)
        result_sens = dcf.calculate(stock)
        if result_sens.fair_value:
            row.append(result_sens.fair_value)
        else:
            row.append(None)
    sensitivity_matrix.append(row)

# 创建 DataFrame
df_sensitivity = pd.DataFrame(
    sensitivity_matrix,
    index=[f'增长 {g}%' for g in growth_rates],
    columns=[f'折现 {r}%' for r in discount_rates]
)

print("\n📊 敏感性分析矩阵 - 内在价值 ($)")
print("=" * 60)
print(df_sensitivity.to_string(float_format='%.2f'))

print(f"\n当前股价：${stock.current_price:.2f}")
print(f"\n💡 分析：")
print(f"   - 增长率越高，估值越高")
print(f"   - 折现率越高，估值越低")
print(f"   - 估值范围：${df_sensitivity.min().min():.2f} - ${df_sensitivity.max().max():.2f}")
```

输出：

```
📊 敏感性分析矩阵 - 内在价值 ($)
============================================================
        折现 8%  折现 10%  折现 12%
增长 5%  146.25  105.71   82.27
增长 8%  176.06  126.22   97.51
增长 11% 211.46  150.45  115.42
增长 14% 253.39  178.99  136.42

当前股价：\$248.80

💡 分析：
   - 增长率越高，估值越高
   - 折现率越高，估值越低
   - 估值范围：\$82.27 - \$253.39
```

![DCF 敏感性分析热力图](https://raw.githubusercontent.com/wangzhe3224/valueinvest-learn/main/learn/dcf_valuation/plots/sensitivity_heatmap.png)

### 如何读这张表

这张矩阵表是投资决策最有用的工具之一：

**估值范围：\$82.27 — \$253.39**

这意味着，即使在"合理"的参数范围内，苹果的内在价值可以从 \$82 到 \$253，相差超过 3 倍！这就是 DCF 的现实——它不能给你一个精确的数字，但能帮你建立一个思考框架。

**与当前股价对比：\$248.80**

当前股价 \$248.80 接近矩阵的右上角（高增长 + 低折现率）。这意味着**市场对苹果的定价隐含了相当乐观的预期**——需要同时满足：
- 前 5 年 14% 的高增长，或
- 8% 增长配合 8% 的低折现率

**怎么用这张表做决策？**

如果你认为苹果未来 5 年能维持 8-11% 的增长，且合理的折现率是 10%，那么合理价格在 **\$126-\$150** 之间。当前 \$248.80 的价格意味着市场比你更乐观。这时你需要思考：市场看到了什么你没看到的？

## 8. Reverse DCF - 市场预期什么？

如果说 DCF 是"我预测未来，算出合理价格"，那 **Reverse DCF 就是反过来——给定当前价格，倒推市场在预期什么。**

这是一种非常实用的思维方式。与其纠结"我的预测对不对"，不如问一个更本质的问题：**市场现在在赌什么？我同不同意这个赌注？**

```python
# Reverse DCF 分析
reverse_result = engine.run_single(stock, "reverse_dcf")

print("\n🔄 Reverse DCF 分析")
print("=" * 60)

if reverse_result.details:
    implied_growth = reverse_result.details.get('implied_growth_1_5', 0)
    implied_growth_6_10 = reverse_result.details.get('implied_growth_6_10', 0)

    print(f"\n当前股价 ${stock.current_price:.2f} 隐含的预期：")
    print(f"   前5年年均增长：{implied_growth:.1f}%")
    print(f"   后5年年均增长：{implied_growth_6_10:.1f}%")

    print(f"\n📝 分析：")
    for analysis in reverse_result.analysis:
        print(f"   • {analysis}")

    # 与历史增长对比
    print(f"\n💡 思考：")
    if implied_growth > 15:
        print(f"   ⚠️  市场预期高增长 ({implied_growth:.1f}%)，需要验证是否可持续")
    elif implied_growth < 5:
        print(f"   市场预期低增长 ({implied_growth:.1f}%)，可能是价值机会或衰退预期")
    else:
        print(f"   市场预期中等增长 ({implied_growth:.1f}%)，相对合理")

    print(f"\n   你需要判断：公司能否实现 {implied_growth:.1f}% 的年增长？")
else:
    print(f"❌ 分析失败：{reverse_result.error}")
```

输出：

```
🔄 Reverse DCF 分析
============================================================

当前股价 \$248.80 隐含的预期：
   前5年年均增长：19.8%
   后5年年均增长：9.9%

📝 分析：
   • Market prices in 19.8% annual growth for years 1-5
   • Years 6-10 growth implied at 9.9%
   • Sensitivity range: 14.8% to 24.8% growth

💡 思考：
   ⚠️  市场预期高增长 (19.8%)，需要验证是否可持续

   你需要判断：公司能否实现 19.8% 的年增长？
```

### 解读 Reverse DCF 结果

**\$248.80 的股价隐含了前 5 年 19.8% 的年增长。** 这是一个相当高的数字。

让我们做一个快速的合理性检查：

- 苹果当前 FCF 为 \$106.31B
- 如果未来 5 年每年增长 19.8%，第 5 年 FCF 将达到约 **\$264B**
- 这意味着苹果需要在现有 \$106B 的基础上，5 年内再增加 \$158B 的自由现金流
- 相当于每年新增约 **\$30B** 的自由现金流——大约是当前 Netflix 整个公司的市值

这个预期是否合理？这取决于你如何看待：
- Apple Intelligence 能否带来新的付费用户和收入？
- iPhone 在 AI 时代的换机周期能否加速？
- 服务业务（App Store、iCloud、Apple TV+）能否持续高增长？

**Reverse DCF 的价值不在于给出一个精确数字，而在于把模糊的"贵不贵"转化为一个具体的问题："公司能不能做到 X% 的增长？"** 这个问题你可以用行业研究、竞争分析和公司基本面来回答。

## 9. A股案例 - 伊利股份

让我们用相同方法分析一只 A 股。选伊利股份的原因是：它是典型的成熟消费品公司，FCF 稳定，适合 DCF 分析。

```python
# 分析 A 股 - 伊利股份 (600887)
ticker_cn = "600887"
print(f"正在获取 {ticker_cn} (伊利股份) 数据...\n")

try:
    stock_cn = Stock.from_api(ticker_cn)

    print("=" * 60)
    print(f"公司：{stock_cn.name} ({stock_cn.ticker})")
    print("=" * 60)
    print(f"当前股价：¥{stock_cn.current_price:.2f}")
    print(f"市值：¥{stock_cn.market_cap/1e8:.2f}亿")
    print(f"自由现金流：¥{stock_cn.fcf/1e8:.2f}亿")
    print(f"每股收益：¥{stock_cn.eps:.2f}")

    # 设置 A 股参数（通常增长率较低，折现率考虑中国市场风险溢价）
    stock_cn.growth_rate_1_5 = 6.0   # A股成熟公司通常增长较慢
    stock_cn.growth_rate_6_10 = 3.0
    stock_cn.terminal_growth = 2.0
    stock_cn.discount_rate = 12.0    # 中国市场风险溢价较高

    # 运行 DCF
    result_cn = engine.run_single(stock_cn, "dcf")

    if result_cn.fair_value:
        print(f"\n💰 DCF 估值结果")
        print(f"   内在价值：¥{result_cn.fair_value:.2f}")
        print(f"   溢价/折价：{result_cn.premium_discount:+.1f}%")
        print(f"   评估：{result_cn.assessment}")
        print(f"   终值占比：{result_cn.details['terminal_value_pct']:.1f}%")
    else:
        print(f"\n❌ 估值失败：{result_cn.error}")

except Exception as e:
    print(f"❌ 获取数据失败：{e}")
    print("\n提示：确保已安装 AKShare: pip install valueinvest[ashare]")
```

输出：

```
正在获取 600887 (伊利股份) 数据...

============================================================
公司：伊利股份 (600887)
============================================================
当前股价：¥26.16
市值：¥1654.71亿
自由现金流：¥71.77亿
每股收益：¥1.65

💰 DCF 估值结果
   内在价值：¥4.71
   溢价/折价：-82.0%
   评估：Overvalued
   终值占比：41.4%
```

### A 股 DCF 的特殊考量

**参数设定的差异**：

| 参数 | 美股（苹果） | A股（伊利） | 原因 |
|------|------------|------------|------|
| 前 5 年增长 | 8% | 6% | 中国乳制品市场已进入存量竞争 |
| 折现率 | 10% | 12% | A 股市场波动更大，要求更高风险溢价 |
| 永续增长 | 2.5% | 2.0% | 更保守的长期假设 |

**关于折现率 12% 的说明**：A 股的折现率通常比美股高 2-3 个百分点，原因包括：
- 市场波动性更大（更高的 $\beta$）
- 政策不确定性更高
- 会计质量和公司治理风险溢价
- 个体投资者占比高导致的价格波动

**结果解读**：DCF 给出的内在价值只有 ¥4.71，远低于当前 ¥26.16 的股价。但**不要直接下结论说伊利被严重高估**。这个结果反映的是：DCF 用的是自由现金流，而伊利的利润和现金流之间存在较大差异（资本支出高、存货和应收账款占用资金）。对于这类"利润好看但现金流紧张"的公司，DCF 往往会给出偏低的估值。这也正是 DCF 的局限性之一。

## 10. DCF 的局限性和最佳实践

### 局限性

**1. 对参数极度敏感**

前面敏感性分析已经展示了：同样的公司，增长率从 5% 变到 14%，估值可以从 \$82 到 \$253。**"Garbage in, garbage out" 是 DCF 最大的标签。**

终值往往占总价值的 50-70%，而终值又高度依赖永续增长率假设。永续增长率从 2% 变到 3%，估值可能变化 20-30%。

**2. 不适用的情况**

DCF 不是万能的。以下情况使用 DCF 需要格外小心：

| 场景 | 为什么不适合 | 替代方法 |
|------|------------|---------|
| 负现金流的公司（早期初创） | 没有正的 FCF 可以折现 | VC 估值法、可比交易法 |
| 周期性行业（钢铁、航运） | 现金流波动极大，无法预测 | 周期调整 PE、NAV 估值 |
| 银行、保险 | 现金流定义完全不同 | P/B、ROE 估值 |
| 重并购公司 | FCF 被并购活动扭曲 | 调整后的 FCF 或分部估值 |
| 高杠杆公司 | 债务结构复杂 | APV（调整现值法） |

**3. 假设外推风险**

DCF 假设公司会按你的预测增长 10 年，然后永远按永续增长率增长。现实中：
- 行业格局可能被颠覆（诺基亚、柯达）
- 管理层可能犯错
- 宏观环境可能剧变
- 技术进步可能改变一切

**巴菲特说得好**："预测未来收益，比预测明天的天气还难。DCF 的价值不在于预测的准确性，而在于强迫你思考企业未来会产生多少现金流。"

### 最佳实践

**1. 保守假设**

> "在估值中，乐观会害死你，保守只会让你错过一些机会。"

- 增长率不超过行业平均或历史平均
- 折现率宁可高一点，不要低估风险
- 永续增长率宁可用 2%，不要用 3%

**2. 敏感性分析（必须做）**

- 测试不同参数组合（至少 3x3 矩阵）
- 得出估值区间而非单一值
- 关注什么参数变化会导致结论翻转

**3. 结合其他方法**

没有任何一种估值方法是完美的。**多种方法交叉验证**是成熟投资者的标志：

| 方法 | 擅长 | 不擅长 |
|------|------|--------|
| DCF | 绝对价值、长期判断 | 短期定价、高波动公司 |
| PE/PB | 快速比较、行业对标 | 周期公司（PE 失真） |
| PEG | 成长性定价 | 亏损公司 |
| EV/EBITDA | 跨公司比较（去除资本结构影响） | 重金融公司 |

**4. 关注 FCF 质量**

DCF 的输入是自由现金流，所以 FCF 的质量直接决定估值的质量。检查：
- FCF 是否稳定（不要用波动极大的年份）
- 资本支出是否合理（是否被人为压低来美化 FCF）
- 经营现金流和净利润的差距（差距大可能意味着应收账款问题）

**5. 终值占比**

- < 50%：非常稳健，预测期内的现金流贡献了大部分价值
- 50-60%：正常水平
- \> 70%：需要警惕——你的估值几乎完全依赖于"公司会永远增长"这个假设

```python
# 综合分析示例
print("\n📊 综合估值分析 - " + stock.name)
print("=" * 60)

# 运行多种估值方法
results_all = engine.run_all(stock)

# 筛选出有效结果
valid_results = [(r.method, r.fair_value, r.premium_discount)
                 for r in results_all if r.fair_value and r.fair_value > 0]

if valid_results:
    df_results = pd.DataFrame(valid_results,
                              columns=['方法', '内在价值', '溢价/折价%'])
    print(df_results.to_string(index=False, float_format=lambda x: f'{x:.2f}'))

    avg_value = np.mean([r[1] for r in valid_results])
    print(f"\n💡 平均估值：${avg_value:.2f}")
    print(f"   当前股价：${stock.current_price:.2f}")
    print(f"   平均溢价/折价：{((avg_value - stock.current_price)/stock.current_price*100):+.1f}%")
```

输出：

```
📊 综合估值分析 - Apple Inc.
============================================================
                 方法   内在价值  溢价/折价%
     Graham Formula 262.02    5.30
      DCF (10-Year) 126.22  -49.30
        Reverse DCF 248.80    0.00
  EPV (Zero Growth)  74.91  -69.90
DDM (Gordon Growth)  37.51  -84.90
      Two-Stage DDM  14.90  -94.00
          PEG Ratio 124.19  -50.10
               GARP 167.50  -32.70
         Rule of 40 248.80    0.00
      P/B Valuation 112.48  -54.80
    Residual Income 736.69  196.10
      Magic Formula  86.35  -65.30
     Owner Earnings  96.25  -61.30
          EV/EBITDA 125.87  -49.40
     Altman Z-Score 248.80    0.00
       SBC Analysis 248.80    0.00
Value Trap Detector 248.80    0.00
  Piotroski F-Score 248.80    0.00
        PE Relative 232.40   -6.60
        PB Relative 274.98   10.50
    Beneish M-Score 248.80    0.00

💡 平均估值：\$200.66
   当前股价：\$248.80
   平均溢价/折价：-19.3%
```

### 多方法对比的启发

这张表展示了 20 种不同估值方法的结果，估值从 \$14.90 到 \$736.69，跨度极大。几个有趣的观察：

1. **Graham Formula (\$262.02) 和 PE/PB Relative (\$232-\$275)** 认为苹果基本合理甚至略微低估。这些方法主要看盈利和账面价值，对成长型公司更友好。

2. **DCF (\$126.22) 和 EV/EBITDA (\$125.87)** 认为苹果被高估约 50%。这些方法更关注现金流，给出的估值更保守。

3. **DDM 方法 (\$14.90-\$37.51)** 给出极低的估值。这是因为苹果的股息率很低（约 0.5%），DDM 假设投资者只通过分红获利，完全忽略了苹果的回购和留存收益再投资。

4. **综合平均 (\$200.66)** 比当前股价低约 19%。这提供了一个"中间地带"——不是所有方法都认为苹果被高估，但多数现金流导向的方法给出了偏保守的结论。

**教训**：不要迷信任何单一方法。把多种方法当作不同的"棱镜"，从不同角度看同一家公司，你的判断才会更全面。

## 11. 总结

### 关键要点

1. **DCF 核心**：公司价值 = 未来现金流的现值。这不是一个公式，而是一种思维方式——投资就是买入未来现金流的索取权。

2. **关键参数**：
   - 增长率（Years 1-5, 6-10）—— 最敏感的参数，需要深入研究
   - 终值增长率（2-3%）—— 代表"到永远"的增长，保守为上
   - 折现率（8-12%）—— 你的机会成本，越高越安全

3. **敏感性分析必不可少**—— 没有做过敏感性分析的 DCF 就像没有刻度的尺子

4. **Reverse DCF 帮助理解市场预期**—— 把"贵不贵"转化为"市场在赌什么"

5. **结合多种估值方法**—— DCF 是重要参考，但永远不是唯一标准

### 实践建议

- 使用 `valueinvest` 库快速获取数据和估值
- 对关键参数进行敏感性分析
- 优先分析 FCF 稳定的成熟公司
- 将 DCF 作为估值的参考之一，而非唯一标准
- **关注过程而非结果**—— DCF 最大的价值在于强迫你思考"这家公司未来能赚多少钱"，这个思考过程本身就让你成为更好的投资者

### 下一步学习

- [Graham Number 估值教程](https://github.com/wangzhe3224/valueinvest-learn) —— 学习 Benjamin Graham 的经典估值方法
- [Owner Earnings 估值教程](https://github.com/wangzhe3224/valueinvest-learn) —— 巴菲特推崇的"所有者盈余"概念
- [ValueInvest 文档](https://github.com/wangzhe3224/valueinvest) —— 了解更多估值工具和方法

---

> 本文所有代码和数据分析均基于 [ValueInvest](https://github.com/wangzhe3224/valueinvest) 开源库，欢迎 Star 和贡献。

```python
# 保存分析报告
from datetime import datetime

report = f"""
DCF 估值分析报告
{'=' * 60}
生成时间：{datetime.now().strftime('%Y-%m-%d %H:%M:%S')}

股票：{stock.name} ({stock.ticker})
当前股价：${stock.current_price:.2f}

DCF 参数：
- 增长率 (Years 1-5): {growth_rate_1_5}%
- 增长率 (Years 6-10): {growth_rate_6_10}%
- 终值增长率: {terminal_growth}%
- 折现率: {discount_rate}%

估值结果：
- 内在价值：${result.fair_value:.2f}
- 溢价/折价：{result.premium_discount:+.1f}%
- 评估：{result.assessment}
- 终值占比：{result.details['terminal_value_pct']:.1f}%
- 置信度：{result.confidence}

Reverse DCF：
- 市场隐含增长 (Years 1-5): {reverse_result.details.get('implied_growth_1_5', 'N/A')}%

分析要点：
{chr(10).join(['- ' + a for a in result.analysis])}
"""

print(report)
print("\n✓ 分析完成！")
print(f"\n📚 更多教程和案例，请访问：https://github.com/wangzhe3224/valueinvest")
```

输出：

```
DCF 估值分析报告
============================================================
生成时间：2026-03-28 22:02:09

股票：Apple Inc. (AAPL)
当前股价：\$248.80

DCF 参数：
- 增长率 (Years 1-5): 8.0%
- 增长率 (Years 6-10): 4.0%
- 终值增长率: 2.5%
- 折现率: 10.0%

估值结果：
- 内在价值：\$126.22
- 溢价/折价：-49.3%
- 评估：Overvalued
- 终值占比：52.3%
- 置信度：Medium

Reverse DCF：
- 市场隐含增长 (Years 1-5): 19.8%

分析要点：
- 10-year DCF with terminal value
- Terminal Value represents 52.3% of total value
- FCF Year 10: 190.05B


✓ 分析完成！

📚 更多教程和案例，请访问：https://github.com/wangzhe3224/valueinvest
```
