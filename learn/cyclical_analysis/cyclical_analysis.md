# 周期股分析：价值投资中最危险的领域

> 本文使用 [ValueInvest](https://github.com/wangzhe3224/valueinvest) 开源库进行数据分析与可视化。所有代码和工具均可在该仓库获取。

---

## 前言：为什么周期股是"价值陷阱"的高发区？

在价值投资的实践中，**周期股**是最容易被忽视、也最容易亏钱的领域。

传统估值方法（PE、DCF）在周期股上不仅失效，还会产生致命误导：**周期高点时PE最低（看起来最便宜），周期低谷时PE最高（看起来最贵）**。如果不理解周期，"低PE买入"可能正好买在最高点——这被称为"PE陷阱"。

许多价值投资者在周期股上栽跟头，不是因为分析不够深入，而是因为**用错了估值框架**。本文将带你从零构建一套完整的周期股分析体系。

## 学习目标

1. 理解为什么传统估值方法在周期股上失效
2. 掌握周期位置的判断方法（多维度评分体系）
3. 学会周期调整后的估值方法（PB、标准化PE、FCF收益率、股息率）
4. 理解A股和美股周期股的不同投资策略
5. 使用 `valueinvest` 库对真实周期股进行完整分析

---

## 一、什么是周期股？

周期股是指盈利随经济周期、行业周期大幅波动的公司。它们的共同特征：

- **盈利波动剧烈**：好年份赚得盆满钵满，差年份直接亏损
- **重资产**：固定资产占比高，产能扩张/收缩有滞后性
- **价格弹性大**：产品价格由供需决定，而非品牌溢价

### 典型周期行业

| 周期类型 | 代表行业 | 代表公司 |
|---------|---------|---------|
| 大宗商品 | 煤炭、钢铁、有色 | 中国神华、宝钢股份 |
| 产能周期 | 化工、水泥、造纸 | 万华化学、海螺水泥 |
| 航运周期 | 集装箱、干散货 | 中远海控、中远海能 |
| 金融周期 | 银行、保险 | 招商银行、中国平安 |
| 地产周期 | 房地产开发 | 万科、保利发展 |
| 能源周期 | 油气、新能源 | 中国石油、隆基绿能 |

**识别周期股的关键**：看过去10年净利润的波动幅度。如果最高年利润是最低年利润的5倍以上，基本可以判定为强周期股。

---

## 二、为什么传统估值方法会失效？

### "PE陷阱"：最危险的估值错觉

周期股最大的陷阱就是**低PE买入**。来看一个典型的10年周期模拟数据：

```
周期位置:  低谷 → 复苏 → 繁荣 → 高峰 → 衰退 → 萧条
股价走势:  低   → 上涨 →  高   →  最高 → 下跌 →  低
每股收益:  低/负 → 恢复 → 极高  →  最高 → 暴跌 → 低/负
PE估值:    高/∞ → 中等 →  低    → 最低  → 升高  → 高/∞
          (便宜?)  (合理)  (便宜!) (捡到?) (变贵)  (昂贵)
```

**关键洞察**：
- 周期**高点**时 PE 最低 → 看起来最便宜 → 实际上最危险
- 周期**低谷**时 PE 最高（甚至为负）→ 看起来最贵 → 实际上是买入机会

让我们用数据直观感受这个陷阱：

```python
import numpy as np
import matplotlib.pyplot as plt

# 模拟一个典型的10年周期
years = np.arange(1, 11)
# 盈利周期：低谷-复苏-繁荣-高峰-衰退-萧条
eps = [0.5, 1.0, 2.0, 3.5, 5.0, 4.0, 2.5, 1.0, 0.3, 0.8]
# 股价提前反应
price = [12, 18, 28, 35, 40, 32, 22, 12, 10, 15]
# PE
pe = [p/e for p, e in zip(price, eps)]

fig, axes = plt.subplots(3, 1, figsize=(12, 9), sharex=True)

colors = ['#e74c3c' if p == max(pe) else '#2ecc71' if p == min(pe) else '#3498db' for p in pe]

axes[0].bar(years, eps, color='#3498db', alpha=0.8)
axes[0].set_ylabel('每股收益 (元)')
axes[0].set_title('周期股的盈利、股价与PE')
axes[0].axvline(x=5, color='red', linestyle='--', alpha=0.5, label='周期高点')
axes[0].axvline(x=9, color='green', linestyle='--', alpha=0.5, label='周期低谷')
axes[0].legend()

axes[1].plot(years, price, 'o-', color='#e67e22', linewidth=2, markersize=8)
axes[1].set_ylabel('股价 (元)')

axes[2].bar(years, pe, color=colors, alpha=0.8)
axes[2].axhline(y=np.mean(pe), color='gray', linestyle=':', alpha=0.7)
axes[2].set_ylabel('PE倍数')
axes[2].set_xlabel('年份')
axes[2].annotate('PE最低=8x\n(周期高点!)', xy=(5, min(pe)), xytext=(6.5, min(pe)+4),
                arrowprops=dict(arrowstyle='->', color='red'), fontsize=10, color='red')
axes[2].annotate('PE最高=33x\n(周期低谷)', xy=(9, max(pe)), xytext=(7.5, max(pe)-5),
                arrowprops=dict(arrowstyle='->', color='green'), fontsize=10, color='green')

plt.tight_layout()
plt.show()
```

![PE陷阱可视化](https://raw.githubusercontent.com/wangzhe3224/valueinvest-learn/main/learn/cyclical_analysis/plots/pe_trap.png)

上图清楚地展示了：**第5年周期高点时PE最低（8x），第9年周期低谷时PE最高（33x）**。如果你用"低PE = 便宜"的逻辑，会在最该卖出的时候买入。

**这就是为什么很多"价值投资者"在周期股上亏大钱**——他们看到了诱人的低PE，却不知道这正是周期顶点的标志。

### DCF 在周期股上的问题

DCF 假设未来盈利**稳定增长**，但周期股的盈利是**脉冲式**的：

| DCF输入 | 周期股现实 | 结果 |
|---------|-----------|------|
| 用高峰期盈利做DCF | 盈利即将暴跌 | 严重高估 |
| 用低谷期盈利做DCF | 盈利即将恢复 | 严重低估 |
| 用平均盈利做DCF | 忽略了周期位置 | 估值模糊 |

---

## 三、周期股的正确估值方法

### 核心原则：先判周期位置，再做估值

周期股估值三步走：

```
第一步: 判断周期位置 → 我们在周期的哪个阶段？
第二步: 选择估值方法 → 根据周期阶段调整估值参数
第三步: 制定投资策略 → 买入/持有/卖出
```

**为什么必须先判周期？** 因为同样的PB=1.0x，在周期低谷时可能是极佳的买入机会，但在周期高位时可能意味着资产质量恶化。脱离周期位置谈估值，就像脱离体温谈健康状况一样没有意义。

### 估值方法选择

| 方法 | 核心逻辑 | 适用场景 | 优势 | 局限 |
|------|---------|---------|------|------|
| **周期调整PB** | PB相对稳定，用历史分位数判断 | 大多数周期股 | 不受盈利波动影响 | 不适用于轻资产公司 |
| **标准化PE** | 用3-5年平均EPS计算PE | 有稳定盈利历史的公司 | 消除单年异常 | 需要足够历史数据 |
| **FCF收益率** | FCF比净利润更难操纵 | 现金流好的公司 | 质量更高 | 资本密集型公司FCF可能为负 |
| **股息率** | 分红是真金白银 | 有稳定分红的公司 | 下行保护 | A股分红文化弱，适用面窄 |

**实际操作中的优先级**：周期调整PB（首选）> 标准化PE > FCF收益率 > 股息率

---

## 四、实战分析：中远海控（601919）

以**中远海控**为例，它是航运周期的典型代表。2021年因集装箱运价暴涨，净利润超过1000亿元；而2019年净利润仅67亿元——**盈利波动超过15倍**。这就是典型的强周期股。

### 4.1 环境准备

```python
import sys
sys.path.insert(0, '/path/to/valueinvest')

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from matplotlib import font_manager

# 设置中文字体
plt.rcParams['font.sans-serif'] = ['Arial Unicode MS', 'PingFang SC']
plt.rcParams['axes.unicode_minus'] = False
```

### 4.2 获取数据

```python
from valueinvest.stock import Stock
from valueinvest.cyclical import (
    CyclicalAnalysisEngine, CyclicalStock, CycleType, CycleStrength,
    CyclePositionScorer, CycleIndicator, IndicatorCategory,
    CyclicalPBValuation, CyclicalPEValuation, CyclicalFCFValuation,
    CyclicalDividendValuation, AShareCyclicalStrategy
)

# 从API获取股票数据
stock = Stock.from_api("601919")
print(f"公司: {stock.name}")
print(f"当前价格: {stock.current_price:.2f} 元")
print(f"PE: {stock.pe_ratio:.2f}x")
print(f"PB: {stock.pb_ratio:.2f}x")
print(f"每股收益: {stock.eps:.4f} 元")
print(f"每股净资产: {stock.bvps:.4f} 元")
```

输出：

```
公司: 中远海控
当前价格: 14.45 元
PE: 7.26x
PB: 0.95x
每股收益: 1.9900 元
每股净资产: 15.1682 元
```

将 Stock 转换为 CyclicalStock，标注周期类型：

```python
cyclical_stock = CyclicalStock.from_stock(
    stock=stock,
    cycle_type=CycleType.SHIPPING,
    cycle_strength=CycleStrength.STRONG
)
```

> **为什么标注周期类型很重要？** 不同周期类型的行业指标、估值阈值、策略参数都不同。航运周期和煤炭周期的"低谷"标准完全不一样，标注周期类型可以让分析引擎使用针对性的参数。

---

## 五、一步完成分析（CyclicalAnalysisEngine）

对于快速获得整体判断，`CyclicalAnalysisEngine` 提供了一站式分析：自动完成周期定位、多维估值、策略生成。

```python
engine = CyclicalAnalysisEngine()
result = engine.analyze(cyclical_stock)
```

### 分析结果总览

```
==================================================
  中远海控 周期股分析报告
==================================================

周期位置: 上行中期
   综合评分: 2.8/5.0
   置信度: Low

投资评级: 谨慎
   总分: 34/100
   风险等级: 中

策略建议: 持有
   目标价格: 22.75 元
   止损价格: 10.62 元
   建议仓位: 5%
   持有周期: 1-3年
```

**结果解读**：

- **周期位置"上行中期"**：航运周期正在从低谷恢复，但还没有到达繁荣阶段。这是一个需要耐心等待的位置——不是最佳买入点，也不是卖出点。
- **评分2.8/5.0，置信度Low**：评分偏低说明当前不是特别有吸引力的位置，置信度低说明数据信号不够强。这提醒我们需要更谨慎地对待这个结论。
- **建议仓位5%**：即使看好，周期股也不应重仓。5%的仓位上限体现了对周期不确定性的尊重。

### 多维度估值结果

```
==================================================
  多维度估值结果
==================================================

【周期调整PB】
  公允价值: 30.34 元 | 当前价格: 14.45 元
  评估: 低估 | 置信度: Low

【标准化PE】
  公允价值: 29.85 元 | 当前价格: 14.45 元
  评估: 低估 | 置信度: Medium

【FCF收益率】
  公允价值: 19.15 元 | 当前价格: 14.45 元
  评估: 低估 | 置信度: Low

【股息率】
  公允价值: N/A | 当前股息率: 0.00%
  评估: N/A（无分红数据）
```

**四种方法的分歧**：
- PB和PE给出较高的公允价值（约30元），暗示有较大上涨空间
- FCF收益率给出较低的公允价值（约19元），更保守
- 股息率方法无法评估，因为当前没有分红

**为什么会出现分歧？** 这正是多维度估值的价值所在。单一方法可能被特定因素扭曲（比如PB没有考虑资产质量，PE没有考虑周期位置），但多种方法交叉验证可以给出更稳健的结论。当多种方法指向同一方向时，置信度更高；当出现分歧时，说明需要更深入的分析。

### 买入条件检查

```
通过: 3/7 (43%)

  ❌ 周期位置       — 当前处于上行中期，不是最佳买入点
  ✅ 估值安全       — PB < 1.0x，低于净资产
  ✅ 资产质量       — 负债率合理
  ❌ 现金流         — FCF质量差
  ❌ 分红能力       — 无分红
  ❌ ROE水平        — 当前ROE偏低
  ✅ 历史PB低位     — PB处于历史较低分位
```

**只有3/7项通过，这明确告诉我们现在不是积极买入的时机。** 等待周期进一步下行到低谷、更多条件满足时再建仓，是更明智的选择。

### 风险与催化剂

**风险因素**：
1. 现金流质量差，盈利可能虚高

**积极催化剂**：
1. 周期即将反转，布局良机
2. PB < 1.0，安全边际极高

---

## 六、手动拆解：周期位置评分

理解了引擎的整体输出后，让我们手动构建周期评分，理解引擎背后的逻辑。

`CyclePositionScorer` 从四个维度评估周期位置：

| 维度 | 权重(A股) | 说明 | 核心指标 |
|------|----------|------|---------|
| 估值指标 | 30% | PB、PE的历史分位数 | PB分位数、PE分位数 |
| 财务指标 | 20% | ROE、利润率、FCF质量 | ROE、毛利率、FCF/净利润 |
| 行业指标 | 35% | 产能利用率、价格趋势 | 运价指数、新船订单/运力 |
| 市场情绪 | 15% | 资金流向、分析师预期 | （本例中暂缺详细数据） |

**为什么行业指标权重最高（35%）？** 因为行业供需关系是驱动周期的核心力量。估值和财务指标更多是"结果"，而行业指标才是"原因"。在航运业，运价指数和新船订单直接决定了未来几年的盈利走势。

```python
from valueinvest.cyclical import (
    CyclePositionScorer, CycleIndicator, IndicatorCategory, MarketType
)

scorer = CyclePositionScorer(market=MarketType.A_SHARE)

# 添加估值指标
scorer.add_indicator(CycleIndicator(
    name="PB分位数", value=cyclical_stock.pb,
    category=IndicatorCategory.VALUATION,
    historical_avg=1.0, percentile=30, trend="stable",
    weight=0.5, unit="x",
    description="当前PB相对历史水平",
    historical_values=cyclical_stock.historical_pb or [cyclical_stock.pb]
))

scorer.add_indicator(CycleIndicator(
    name="PE分位数", value=cyclical_stock.pe,
    category=IndicatorCategory.VALUATION,
    historical_avg=7.3, percentile=25, trend="down",
    weight=0.5, unit="x",
    description="当前PE相对历史水平",
    historical_values=cyclical_stock.historical_pe or [cyclical_stock.pe]
))

# 添加财务指标
scorer.add_indicator(CycleIndicator(
    name="ROE", value=cyclical_stock.roe * 100,
    category=IndicatorCategory.FINANCIAL,
    historical_avg=12.0, percentile=40, trend="stable",
    weight=0.4, unit="%",
    description="净资产收益率",
    historical_values=cyclical_stock.historical_roe or [cyclical_stock.roe * 100]
))

scorer.add_indicator(CycleIndicator(
    name="毛利率", value=cyclical_stock.gross_margin * 100,
    category=IndicatorCategory.FINANCIAL,
    weight=0.3, unit="%",
    description="毛利率",
    historical_values=[cyclical_stock.gross_margin * 100]
))

scorer.add_indicator(CycleIndicator(
    name="FCF/净利润", value=cyclical_stock.fcf_to_net_income * 100,
    category=IndicatorCategory.FINANCIAL,
    weight=0.3, unit="%",
    description="现金流质量",
    historical_values=[cyclical_stock.fcf_to_net_income * 100]
))

# 添加行业指标
scorer.add_indicator(CycleIndicator(
    name="运价指数", value=1200,
    category=IndicatorCategory.INDUSTRY,
    historical_avg=1500, percentile=35, trend="down",
    weight=0.5, unit="点",
    description="SCFI集装箱运价指数",
    historical_values=[1200, 1500, 2000, 3500, 4000, 3000, 1800]
))

scorer.add_indicator(CycleIndicator(
    name="新船订单/运力", value=8.5,
    category=IndicatorCategory.INDUSTRY,
    historical_avg=10.0, percentile=40, trend="down",
    weight=0.5, unit="%",
    description="新船订单占现有运力比例",
    historical_values=[8.5, 10.0, 12.0, 15.0]
))

cycle_score = scorer.calculate_score()
```

### 评分结果

```
==================================================
  周期位置评分结果
==================================================

周期阶段: 上行中期
   综合评分: 2.53/5.0
   置信度: Low
   评估: 上行初期，逐步建仓

--- 各维度评分 ---
   估值维度: 2.00/5.0
   财务维度: 3.00/5.0
   行业维度: 2.50/5.0
   情绪维度: 3.00/5.0
```

![周期评分雷达图](https://raw.githubusercontent.com/wangzhe3224/valueinvest-learn/main/learn/cyclical_analysis/plots/cycle_radar.png)

**各指标详细状态**：

```
[██░░░] PB分位数: 2.0/5.0 (偏低)
     当前值: 1.0x | 历史均值: 1.0x | 分位: 30%
[██░░░] PE分位数: 2.0/5.0 (偏低)
     当前值: 7.3x | 历史均值: 7.3x | 分位: 25%
[███░░] ROE: 3.0/5.0 (中等)
     当前值: 0.0% | 历史均值: 12.0% | 分位: 40%
[███░░] 毛利率: 3.0/5.0 (中等)
     当前值: 0.0% | 历史均值: 0.0% | 分位: 50%
[███░░] FCF/净利润: 3.0/5.0 (中等)
     当前值: 66.5% | 历史均值: 0.0% | 分位: 50%
[██░░░] 运价指数: 2.0/5.0 (偏低)
     当前值: 1200.0点 | 历史均值: 1500.0点 | 分位: 35%
[███░░] 新船订单/运力: 3.0/5.0 (中等)
     当前值: 8.5% | 历史均值: 10.0% | 分位: 40%
```

**评分解读**：

- **估值维度最低（2.0/5.0）**：PB和PE都处于历史偏低位置，这是积极信号——说明市场对航运业的预期不高，估值有安全边际
- **行业维度偏低（2.5/5.0）**：运价指数处于历史35%分位且趋势向下，新船订单也在减少——行业仍在筑底过程中
- **财务和情绪维度中等**：没有明显的恶化信号，但也谈不上强劲

**综合来看，当前处于"估值便宜但行业未回暖"的阶段**。这就像是冬天即将结束、春天还没到来的时刻——温度很低（估值低），但你还需要等花开（行业回暖）。

---

## 七、手动拆解：四种估值方法

### 7.1 周期调整PB估值

PB是周期股最可靠的估值指标，因为：
- 净资产相对稳定，不像盈利那样大幅波动
- 周期低谷时PB最低，是较好的买入信号
- 不同周期阶段使用不同的PB阈值

```python
pb_valuation = CyclicalPBValuation()
pb_result = pb_valuation.calculate(cyclical_stock)
```

```
当前PB: 0.95x | 每股净资产: 15.17 元
公允价值: 30.34 元
评估: 低估 | 建议操作: 强力买入 | 置信度: Low

估值细节:
  fair_pb: 2.0x（公允PB倍数）
  buy_threshold: 1.5x
  hold_threshold: 2.0x
  sell_threshold: 2.5x
  cycle_phase: mid_upside（上行中期）
```

**PB估值的逻辑**：根据周期阶段调整公允PB。上行中期给予2.0x的公允PB，意味着认为中远海控的净资产在正常周期中应该值2倍。当前PB仅0.95x，远低于公允值，因此显示低估。

**但要注意置信度是Low**——这意味着这个结论需要更多数据验证。PB低估可能有其原因（比如资产质量恶化、运力过剩等），不能仅仅因为PB低就认为便宜。

### 7.2 标准化PE估值

用过去3-5年的平均EPS来计算PE，消除单年异常盈利的影响。

```python
pe_valuation = CyclicalPEValuation()
pe_result = pe_valuation.calculate(cyclical_stock)
```

```
当前PE: 7.26x | 每股收益: 1.99 元
公允价值: 29.85 元
评估: 低估 | 建议操作: 强力买入 | 置信度: Medium

估值细节:
  fair_pe: 15.0x（公允PE倍数）
  buy_threshold: 12.0x
  sell_threshold: 20.0x
```

**标准化PE的逻辑**：给予15x的公允PE（考虑了周期行业的合理估值倍数），当前PE仅7.26x，低于买入阈值12x，因此显示低估。置信度Medium，比PB方法稍高，因为PE数据相对更可靠。

### 7.3 FCF收益率估值

自由现金流比净利润更难操纵，对周期股尤为重要。

```python
fcf_valuation = CyclicalFCFValuation()
fcf_result = fcf_valuation.calculate(cyclical_stock)
```

```
FCF收益率: 9.28% | FCF/净利润: 0.67x
公允价值: 19.15 元
评估: 低估 | 建议操作: 卖出 | 置信度: Low

估值细节:
  fair_fcf_yield: 7.0%（公允FCF收益率）
  fcf_quality: Poor（现金流质量差）
  buy_threshold: 10.0%（FCF收益率>10%才建议买入）
  sell_threshold: 5.0%
```

**FCF估值的警示信号**：虽然显示低估，但有两个值得注意的问题：
1. **FCF/净利润仅0.67x**：意味着每赚1元净利润，只有0.67元的自由现金流。对于航运这种重资产行业，资本开支很大，盈利的"含金量"打了折扣
2. **建议操作却是"卖出"**：这与"低估"的评估看似矛盾，但实际上是因为FCF收益率（9.28%）虽然高于公允水平（7%），但尚未达到买入阈值（10%），且现金流质量被判定为Poor

### 7.4 股息率估值

分红是真金白银，周期股在盈利高峰期通常有丰厚的分红，可以作为安全边际的参考。

```python
div_valuation = CyclicalDividendValuation()
div_result = div_valuation.calculate(cyclical_stock)
```

```
股息率: 0.00% | 分红率: 0.00%
连续分红年数: 0 年
评估: N/A（无分红数据，无法评估）
```

**A股周期股的普遍问题**：分红不稳定是常态。中远海控在2021年暴利年份曾大额分红，但当前无分红。这说明股息率方法在A股周期股上的适用性有限——这也是为什么A股策略更偏向"交易型"而非"分红防御型"。

---

## 八、四种估值方法对比

```python
methods = {
    '周期调整PB': pb_result,
    '标准化PE': pe_result,
    'FCF收益率': fcf_result,
    '股息率': div_result,
}
```

| 方法 | 公允价值 | 溢价/折价 | 评估 | 置信度 |
|------|---------|----------|------|-------|
| 周期调整PB | 30.34元 | +109.9% | 低估 | Low |
| 标准化PE | 29.85元 | +106.6% | 低估 | Medium |
| FCF收益率 | 19.15元 | +32.5% | 低估 | Low |
| 股息率 | N/A | N/A | N/A | N/A |
| **综合估值** | **19.83元** | **+37.3%** | | |

![多维度估值对比](https://raw.githubusercontent.com/wangzhe3224/valueinvest-learn/main/learn/cyclical_analysis/plots/valuation_comparison.png)

**对比分析**：

1. **PB和PE估值一致**（约30元）：这两种方法给出相近的公允价值，增加了结论的可信度
2. **FCF收益率更保守**（19.15元）：它考虑了现金流质量，给出了更低的估值。在实际投资中，保守估计往往更安全
3. **综合估值约19.83元**：剔除股息率（N/A）后取平均。当前价格14.45元，综合来看有约37%的上涨空间
4. **所有方法的置信度都不高**：这再次提醒我们，当前数据信号不够强，需要等待更多确认

---

## 九、投资策略生成

A股周期股策略偏向**交易型**：利用周期波动获取超额收益。与美股周期股策略（偏向分红防御型）有本质区别。

```python
strategy = AShareCyclicalStrategy()
recommendation = strategy.generate_recommendation(cyclical_stock, result)
```

```
==================================================
  A股周期股投资策略
==================================================

操作建议: 持有
   策略类型: 周期交易型

   目标价格: 22.75 元
   止损价格: 10.62 元
   建议仓位: 5%
   持有周期: 1-3年

--- 投资依据 ---
  • 周期阶段：上行中期
  • PB 0.95x，低于净资产，安全边际高
  • 负债率 41.4%，财务健康
  • 持有：等待周期进一步明朗
```

### 买入/卖出清单

**买入条件**（通过率43%）：

| 条件 | 状态 | 说明 |
|------|------|------|
| 周期位置 | ❌ | 当前上行中期，不是最佳买入点 |
| 估值安全 | ✅ | PB < 1.0x，安全边际充足 |
| 资产质量 | ✅ | 负债率合理 |
| 现金流 | ❌ | FCF质量差 |
| 分红能力 | ❌ | 无分红 |
| ROE水平 | ❌ | ROE偏低 |
| 历史PB低位 | ✅ | PB处于历史较低分位 |

**卖出条件**：

| 条件 | 状态 |
|------|------|
| 周期位置 | — |
| 估值过高 | — |
| ROE恶化 | ⚠️ 已触发 |
| 行业信号 | — |
| 市场情绪 | — |

> **注意**：ROE恶化已经触发卖出预警。这是一个重要的风险信号——如果ROE持续恶化，即使周期位置不极端，也应该考虑减仓。

---

## 十、多公司对比

用不同行业的周期股做对比，理解不同周期类型的特征。我们选取了航运、煤炭、化工三个典型周期行业：

```python
tickers = [
    ("601919", "中远海控", CycleType.SHIPPING, "航运"),
    ("601088", "中国神华", CycleType.COMMODITY, "煤炭"),
    ("600309", "万华化学", CycleType.CAPACITY, "化工"),
]
```

| 公司 | 行业 | 价格 | PB | PE | 周期阶段 | 综合评分 | 评级 |
|------|------|------|-----|-----|---------|---------|------|
| 中远海控 | 航运 | 14.45 | 0.95x | 7.26x | 上行中期 | 34 | 谨慎 |
| 中国神华 | 煤炭 | 46.98 | 2.49x | 17.66x | 上行中期 | 69 | 推荐 |
| 万华化学 | 化工 | 87.65 | 2.46x | 73.66x | 上行中期 | 59 | 中性 |

![多公司对比](https://raw.githubusercontent.com/wangzhe3224/valueinvest-learn/main/learn/cyclical_analysis/plots/multi_company_comparison.png)

**对比分析**：

1. **中国神华评分最高（69分，推荐）**：作为煤炭龙头，神华的优势在于盈利稳定性更强——煤炭需求相对刚性，且公司有铁路、电力等一体化业务平滑周期波动。PB 2.49x虽然不低，但与其资产质量匹配。

2. **万华化学中性（59分）**：化工行业的技术壁垒较高，万华作为MDI龙头有一定定价权，但当前PE 73.66x偏高，说明市场已经给予了较高的成长预期。

3. **中远海控评分最低（34分，谨慎）**：航运业的周期性最强（盈利波动15倍以上），且当前现金流质量和ROE都不理想。但PB < 1.0x提供了较好的安全边际。

**给投资者的启示**：同样是周期股，不同行业的投资逻辑差异很大。煤炭股可以当做"高分红的价值股"来持有，而航运股则更适合做波段交易。

---

## 十一、A股 vs 美股周期股策略差异

| 维度 | A股策略 | 美股策略 |
|------|---------|---------|
| 投资目标 | 交易型，追求超额收益 | 分红防御型，稳定收入 |
| 持有周期 | 1-3年 | 5-10年 |
| 目标收益 | 50-200% | 6-10%/年 |
| 单只仓位上限 | 10% | 8% |
| 周期股总仓位上限 | 35% | 25% |
| 核心指标 | PB分位数、周期位置 | 股息率、FCF覆盖率 |
| 买入条件 | 周期低谷 + PB < 1.2x | 连续分红5年 + FCF > 1.5x |
| 卖出信号 | 周期高点 + PB > 2.5x | 削减分红或FCF恶化 |

### 核心区别解读

- **A股周期股**：利用周期波动做波段，在低谷买入、高峰卖出，追求高绝对收益。这与A股波动大、个人投资者占比高、T+1交易制度有关
- **美股周期股**：通过分红获取稳定现金流，长期持有优质周期龙头，追求总回报。这与美股机构投资者占比高、分红文化强、税收优惠（合格分红税率更低）有关

**如果你同时投资A股和美股的周期股，需要切换思维模式**：不要用A股的交易思维去买卖美股周期股，也不要用美股的长期持有思维去应对A股周期股。

---

## 十二、总结与要点

### 周期股投资的核心原则

1. **先判周期，再做估值** — 不确定周期位置之前，所有估值都是空中楼阁
2. **低PE是陷阱，不是机会** — 周期高点PE最低，这是最危险的买入信号
3. **PB比PE更可靠** — 净资产比盈利稳定得多，用PB的历史分位数判断
4. **在别人恐惧时贪婪** — 周期低谷时PE最高、市场最悲观，反而是买入机会
5. **设定纪律性止盈** — 周期高点时不要贪心，严格执行卖出纪律

### 估值方法优先级

```
周期调整PB（首选）> 标准化PE > FCF收益率 > 股息率
```

### 常见错误

- ❌ 用高峰期盈利做DCF → 严重高估
- ❌ 因为"低PE"买入 → 买在最高点
- ❌ 用成长股的估值框架 → 忽略周期性
- ❌ 不设止盈 → 坐过山车

### 最后提醒

周期股投资本质上是**对行业供需拐点的判断**，而不是对财务数据的简单比较。即使最好的分析工具，也只能帮你提高概率，不能消除不确定性。保持纪律、控制仓位、承认自己可能判断错误——这三点比任何估值模型都重要。

---

> 本文使用 [ValueInvest](https://github.com/wangzhe3224/valueinvest) 开源库，所有代码和工具均可在该仓库获取。
