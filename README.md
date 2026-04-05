# ValueInvest Learn

基于 [ValueInvest](https://github.com/wangzhe3224/valueinvest) 库的价值投资学习教程合集。通过 Jupyter Notebook 和实战案例，系统学习股票估值与分析方法。

## 学习材料

| 教程 | 主题 | 核心内容 |
|------|------|----------|
| [DCF 估值](learn/dcf_valuation/dcf_valuation_tutorial.md) | 现金流折现模型 | DCF 原理、参数设定、敏感性分析、Reverse DCF、多股实战 |
| [Graham Number](learn/graham_number/graham_number_tutorial.md) | 格雷厄姆估值法 | Graham Number 公式、Graham Formula、防御型投资者策略、多公司对比 |
| [Owner Earnings](learn/owner_earnings/owner_earnings_tutorial.ipynb) | 所有者盈余 | 巴菲特盈利质量指标、OE 公式拆解、Earnings Quality 评估、估值应用 |

## 推荐学习路径

```
Graham Number → DCF 估值 → Owner Earnings
   (基础)          (核心)        (进阶)
```

1. **Graham Number** — 理解格雷厄姆的价值投资基础，学会快速判断股票是否低估
2. **DCF 估值** — 掌握现金流折现的核心框架，理解增长率和折现率的影响
3. **Owner Earnings** — 从盈利质量角度审视公司，识别净利润的"含水量"

## 环境要求

- Python 3.11+
- Jupyter Notebook / JupyterLab
- [valueinvest](https://github.com/wangzhe3224/valueinvest) 库

```bash
pip install -e ".[learn,fetch]"
```

## 数据源

- **美股**: yfinance（免费，无需 API Key）
- **A股**: AKShare（免费，无需 API Key）
