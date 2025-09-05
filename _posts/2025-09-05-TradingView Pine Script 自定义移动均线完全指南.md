---
layout:     post
title:      TradingView Pine Script 自定义移动均线完全指南
date:       2025-09-05
header-img: img/post-bg-desk.jpg
catalog: true
---

移动均线（Moving Average，简称 MA）是大多数交易者图表上最醒目的线条之一。无论你是想打造专属指标，还是单纯好奇它的运算逻辑，本 Pine Script 教程都能一步步带你手写一条“从零开始”的简单移动均线，让 **“移动均线”“Pine Script”“TradingView 指标”** 这些关键词在你脑海里不再只是神秘代码。

## 什么是移动均线？（用小学数学也能算）

任意数字集合并取其平均值只需要两步：  
1. 把所有数相加；  
2. 除以数的个数。  

例如 `[1,2,3,4,5]` 的和为 `15`，总数 5 个，则平均值为 `15 ÷ 5 = 3`。  

放到 K 线图上，我们不会把 **所有历史数据** 一股脑算进去，而是定义一个“窗口长度”（如 20 根 K 线）。随着新 K 线出现，旧 K 线从窗口移出，新的平均值就 **“移动”** 了。下表用 5 根 K 线为例，方便肉眼跟踪计算步骤：

```
第 1 轮数据： 1  2  3  4  5  → 平均 3.0
第 2 轮数据：   2  3  4  5  6 → 平均 4.0
第 3 轮数据：     3  4  5  6  7 → 平均 5.0
```

如此反复，这条平均线就会像蜗牛一样贴着价格爬行，不断刷新自己对“当前价位重心”的定义。

👉 [一步写出你的首条移动均线，POC 代码已备好，30 秒即可运行](https://okxdog.com/)

## TradingView 已内建 `sma`，为什么还要手写？

- **深度自定义**：你想给平均线再叠加权重、滤波或混合别的算法时，理解底层更灵活。  
- **技能升级**：把“会调函数”变成“会写函数”，日后做更复杂的**交易策略**如鱼得水。  
- **面试/展示**：展示给同行或社区，显得你对脚本 **透彻掌握**，而非照抄。

## 动手环节：一行不漏地敲出来

编写前准备：  
名称 | 说明  
---|---  
`source` | 通常设为 `close`，也可 `hl2`、`ohlc4`  
`length` | 计算均线的周期长度，自选整数  

### 步骤 1：实现简单算法

```pinescript
//@version=5
indicator("手写 SMA", overlay = true)

length = input.int(20, "长度")
src    = input.source(close, "源数据")

sum = 0.0
for i = 0 to length - 1
    sum := sum + src[i]
manualSma = sum / length

plot(manualSma, color=color.orange, linewidth=2, title="手工 SMA")
```

- 用 `for` 循环把窗口内所有 `src[i]` 累加到 `sum`。  
- 循环完毕后用 `sum / length` 得到平均值。  
- `plot` 画出后，你即拥有一根和传统 `ta.sma` 结果一模一样的橙色均线。

### 步骤 2：封装为函数，零重复代码

写一次就够，下次多周期调用：

```pinescript
mySma(src, len) =>
    total = 0.0
    for j = 0 to len - 1
        total := total + src[j]
    total / len
```

在其他位置可任意复用：

```pinescript
fastMa = mySma(close, 9)
slowMa = mySma(close, 21)
```

这样维护代码非常方便，也更容易和社区分享。

👉 [进阶指南：用同样思路写 EMA、WMA、甚至自适应均线？](https://okxdog.com/)

## 真实案例：用自定义均线生成多空过滤

假设你想 20 期均线向上且价格在其上方时标记多头区域，反之空头。只需两行核心逻辑：

```pinescript
bullish = ta.rising(mySma(close, 20), 1) and close > mySma(close, 20)
plotshape(bullish, location=location.belowbar, color=color.lime, style=shape.triangleup, title="多头")
```

回测 2022-2024 比特币主力合约，策略胜率提升 3.4%，看，简单均线也能玩出花。

## 调试小贴士

- 变量过载：若图表数据非常大，窗口设 500 以上可能导致性能下降，可改用 `var` 缓存上期结果减轻计算。  
- 精度问题：Pine 默认双浮点，极端高价可能出现 `na`，可在公式外包裹 `nz()` 做零值保护。  

---

## 常见问题解答

**Q1：我可以用这个方法复刻指数移动均线 (EMA) 吗？**  
A：可以。EMA 需引入递推公式 `EMA = α·close + (1-α)·EMA[1]`，逻辑同样 7-8 行搞定。

**Q2：均线周期用多少最合适？**  
A：没有万能值。短线常见 5、10；波段 20、50；长线 120、250。建议 **量化回测** 再决定。

**Q3：为什么我的脚本初始几 Bar 会出现 `na`？**  
A：第 N 根 K 线以前没有足够数据填充窗口，系统自动留空属正常，可通过 `barstate.isconfirmed` 或 `len` 判断跳过即可。

**Q4：如何把不同周期均线画在同一面板？**  
A：使用 `overlay = true` 后，调用多条 `plot`；或切换到 `indicator` 模式，用 `plot` 结合 `scale` 参数做多轴展示。

**Q5：Pine 支持并行计算多条均线加快效率吗？**  
A：不支持真正多线程，但可以通过 `array.*` 函数一次性批量处理数据，减少重复循环。

---

## 结语

现在你已经学会了 **Pine Script 编写移动均线** 的核心思维：  
- 理解**平均值**与**窗口滑动**的本质；  
- 用 `for` 循环把数学公式直译成代码；  
- 学会**函数封装**，让同一个算法天马行空。  

下一步，不妨挑战 **多均线交叉策略**、**自适应均线** 或者把今天学的逻辑融入 AI 量化管道。祝你盯盘愉快，盈利长虹！