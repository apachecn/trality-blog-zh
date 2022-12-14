# 机器人的诞生:介绍-战略建设基础(第一部分)

> 原文：<https://www.trality.com/blog/strategy-building-basics-pt-i/>

在最近的 [在 Trality](/blog/developing-simple-trading-bot-with-trality-bot-code-editor) 的博客文章中，我们已经看到了在比特币上应用趋势跟踪信号如何在市场低迷时期提供保护。对于当前的系列，我们现在要后退一步，仔细看看将交易策略从一个想法变成一个可部署的机器人所必须考虑的各个部分。

这篇介绍性文章简要描述了策略开发的主要组成部分，即全域选择、时间框架和交易频率、信号选择、参数化、头寸规模和最后的回溯测试。这些构建模块中的每一个都回答了关于我们最终战略的重要问题。

## 投资领域

我们机器人的第一块积木是它将交易的宇宙。投资领域由一组满足一些常见选择标准的预定义的可交易资产组成。在上面提到的帖子中，我们使用了 BTC 作为我们机器人交易的唯一硬币。我们现在更进一步，为这个策略扩展我们的领域。我们这样做的原因是[多样化](https://www.investopedia.com/terms/d/diversification.asp)。“不要把所有的鸡蛋放在一个篮子里”这句老话同样适用于密码交易。这种说法背后的直觉是，平衡投资组合的风险小于其所有单个证券的风险之和。对于给定的预期回报，风险最小的投资组合被认为是最有效的。正如诺贝尔奖获得者哈里·马科维茨(Harry Markowitz)已经指出的那样，“多样化是投资中唯一的免费午餐”。

## 交易间隔

宇宙的选择也受到我们策略的时间框架和交易频率的影响。交易频率越高，单个硬币的流动性、买卖差价和交易成本等因素就变得越重要。(市场-) [流动性](https://www.investopedia.com/terms/l/liquidity.asp)指的是一个人在给定的时间段内，在不影响市场价格的情况下，买卖一项投资的难易程度。例如，如果我们部署一个每周只交易一次的策略，我们不会太在意是否需要等待一段时间才能在市场上获得所需的交易量，而不会因为交易规模而获得糟糕的价格。然而，如果我们每分钟都交易，低流动性会立即使我们的策略无利可图。尤其是在加密交易中，不同资产的流动性差异很大。

此外，我们交易越多，积累的交易成本就越高。一般来说，我们可以观察到任何资产的流动性和交易成本之间的反比关系。由于这些原因，交易频率对策略的成功起着关键的作用。

## 信号

我们策略的核心是选择交易信号。交易信号为我们的交易算法创建了仓位入口和出口。有各种方法可以得出这样的交易信号。一个非常流行的选择是使用[技术交易指标](https://www.investopedia.com/terms/t/technicalindicator.asp)。或者，可以使用机器学习来生成信号。例如，在 Trality 最近的博客文章中，比较了两个指数移动平均线(指标),以生成策略的进场和出场信号。为了避免混淆，我们随后将把信号发生器称为产生进入和退出信号的方法。

无论我们使用什么样的信号生成过程，我们都必须决定我们的策略要基于哪些数据。我们使用价格数据、订单数据还是两者的组合？我们是否使用额外的替代数据？最常见的是，价格数据是按时间间隔(例如每小时、每天等)获取的。)并包含资产的开盘价、最高价、最低价和收盘价(OHLC)以及在此区间内的相关交易量。在金融行话中，这些数据点也被称为蜡烛。[订单簿](https://academy.binance.com/glossary/order-book)包括给定时间点的所有未结订单。通过分析订单数据，我们可以更深入地了解市场。由于订单簿数据的瞬时变化，分析也变得更加复杂。在这份[摩根大通报告](https://faculty.sites.uci.edu/pjorion/files/2018/05/JPM-2017-MachineLearningInvestments.pdf)中可以找到金融替代数据的详细概述。

此外，我们需要决定我们的机器人属于哪一类交易策略。例如，它是采用趋势跟踪策略，还是将其头寸建立在均值回归的基础上？当然，这些不是交易策略的唯一类型，还有很多可以考虑的，我们将在以后的文章中讨论其中的一些。

也有可能(甚至推荐！)在交易策略中使用多个信号发生器。这使得我们的机器人在不同的市场条件下表现良好，并增加了另一层多样化。使用多种信号降低了我们策略的整体波动性，并防止在回溯测试中[过度拟合](https://www.davidhbailey.com/dhbpapers/overfit-tools-at.pdf)。

## 参数化

一旦我们决定使用哪个信号发生器，我们需要为它们指定合适的参数。在技术指标领域，我们经常需要指定回顾期，从而对参数进行加权。在机器学习领域，我们指定了模型的[超参数](https://en.wikipedia.org/wiki/Hyperparameter_(machine_learning))。

例如，在我们之前的文章中，我们使用了两个 EMA 指标，基于 20 小时和 40 小时的时间段。如何选择这些参数本身就是一门艺术。基于历史模拟选择参数时经常出现的一个问题是过拟合。当我们的信号发生器使用许多可以调整的参数时，这个问题最常出现。在这种情况下，调整参数以使它们在过去的数据上表现非常好的诱惑非常大。因此，需要确保参数集不仅对于历史数据是最优的，而且在实际交易策略时继续提供性能。

## 位置尺寸

最后但同样重要的是，为了完成我们的策略，我们需要决定交易多少(仓位大小)。头寸规模处理的是有多少资金被分配到一个交易、头寸甚至策略本身的问题。有各种各样的技术，如固定金额、等百分比、基于风险的头寸规模等等。

在我们之前建立的 BTC 策略中，我们每笔交易使用了 80%的资本。这个数字怎么选？多枚硬币交易的策略怎么样？我们是在所有硬币之间平均分配我们的资本，还是根据信号强度、基础硬币的波动性或总体市场环境等因素改变头寸规模？这一决策会对战略的整体盈利能力产生重大影响。

## 回溯测试

回溯测试是交易策略的事后模拟。在传统意义上，交易策略是根据历史市场数据来评估的。运行回测时，将可用于回测的时间段分为[样本内和样本外数据](https://alvarezquanttrading.com/blog/in-sample-and-out-of-sample-testing/)尤为重要。我们使用样本内数据来优化我们的策略，一旦我们满意了，我们就使用样本外数据来验证我们的结果，并确保我们不会以一个在实际交易中可能表现不佳的过度拟合策略而告终。幸运的是，Trality 回溯测试模块让这变得非常简单！

我们战略发展系列的第一部分到此结束，因为我们已经对战略发展的各个组成部分有了一个大致的了解。请继续关注下一篇文章，在那里第一次学习将被应用于构建我们的多硬币交易机器人的第一次迭代！

现在是时候深入探讨我们战略构建基础系列的第二部分的主题了。

<button type="button" class="chakra-button css-1hnfsz">Go to Pt. 2</button>

* * *

Stefan 是一名 quant，在金融行业有超过 6 年的工作经验。他目前在一家专门为机构客户提供保险投资的独立投资管理公司担任风险管理&分析总监。7 年来，他一直在业余时间开发和实施跨多种资产类别的算法交易策略。