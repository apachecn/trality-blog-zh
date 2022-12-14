# 优化加密交易策略指南

> 原文：<https://www.trality.com/blog/optimizing-crypto-trading-strategies/>

你可能会认为，一旦你确定了适合你个人投资目标的独特交易策略，艰苦的工作就结束了。经验丰富的交易者会告诉你，这与事实相差甚远。加密交易机器人是自动化的，但它们不是自动的，这意味着它们需要定期监控，尤其是在波动性增加的时期。作为一名交易者，你想让你的交易策略与特定的市场机制相匹配，以优化收益，或在低迷时期限制损失，所以我们根据一系列可能的市场条件，整理了一些调整策略的想法。

## **减少大额提款的想法**

使用三重屏障方法管理每次交易的固定风险订单。三重屏障通过包含止损来限制损失，但也在一定时间后平仓，以防止市场改变方向。事实上，我们已经写了一篇内容丰富的 [关于如何退出交易](/blog/closing-trades)即平仓的指导。我们使用 [Trality 代码编辑器](https://www.trality.com/creator/code-editor)将所有东西放在一个简单的交易策略中。所有 Python 代码都以循序渐进的方式进行了解释。

## **使用跟踪止损，让赢家跑起来**

使用跟踪止损允许趋势跟踪策略保持盈利，同时限制最大损失。有关[尾随订单类型](https://docs.trality.com/trality-code-editor/api-documentation/order/creation/trailing-iftouched)的更多信息，请参见 Trality 文档。

```py
""" Stop price is the inital price of the stop order
    For a sell the stop should be placed below the current market price
    When the price moves up by trailing_precent the stop is moved
"""
pos = query_open_position_by_symbol(symbol=data.symbol, include_dust=False)
if pos is not None:
    stoploss_amount = -pos.exposure  # negative sign to sell position
  	stop_price = data.close[-1]*0.975
	  state.sl = order_trailing_iftouched_amount(symbol=data.symbol,  
	                                             amount=stoploss_amount, 
	                                             trailing_percent=0.025, 
	                                             stop_price=stop_price) 
```

下面是 1h 均线交叉策略的回溯测试结果，分别是 10，50 周期和 1d 超级趋势。当快速均线高于慢速均线时开仓，当快速均线低于慢速均线时平仓。即使在上涨的市场中，这种策略也表现不佳。

![](img/29f8670a331ebf86eef5cef7f1d51984.png)![backesting, crypto, BTC, Bitcoin](img/cc267cae7e244ad79b70c6e369b49da5.png)



Backtest results of a 1h EMA crossover strategy



下面是相同的策略，但是仓位退出使用跟踪止损，位于市场下方 2.5%，以 2.5%的增量退出仓位。无论是从利润还是从最大提取额来看，业绩都要好得多。

![](img/29f8670a331ebf86eef5cef7f1d51984.png)![crypto, BTC, Bitcoin, trailing stop loss](img/ac413a99e9b0eb5743f2b47d3694ae6a.png)



1h EMA crossover strategy using a trailing stop-loss



## **修复连续亏损的交易**

有一种方法可以改善一个有赢有输的策略，那就是考虑在最近几次止损后增加一个冷静期来防止交易。策略通常针对特定的市场条件，如趋势或范围。当策略运行良好，这是一个信号，它是“超过黄金”,可能会有很高的胜率。趋势跟踪策略在牛市中可能有 80%的成功率，但在熊市中只有 20%。通过监控最近的胜率，我们可以推断市场条件对 bot 来说不是最佳的，最好等到市场条件看起来更有希望的时候。对于一个在正确的市场条件下预期胜率为 80%的机器人来说，只有 4%的机会连续两次失败。这个信息对决定如何交易下一个信号很有用。

```py
################################################################################
# compute time as a datetime and store
################################################################################
now = datetime.fromtimestamp(data.times[-1] / 1000.0, pytz.UTC)
if state.cooldown[data.symbol] is None:
    state.cooldown[data.symbol] = now

################################################################################
# set a cooldown timer if the stop loss is filled
################################################################################
if sl_order is not None:
    sl_order.refresh()   # make sure the order is up to date
    if sl_order.is_filled():
        # wait before placing more orders if the strategy was stopped out
        state.cooldown[data.symbol] = now + timedelta(hours=COOLDOWN_HOURS)
        log(f"stop loss hit waiting {COOLDOWN_HOURS} hours before trading again", severity=3)

################################################################################
# check before trading if the strategy is in cooldown 
################################################################################
if now >= state.cooldown[data.symbol]:
    ... # strategy code here
    if should_trade:
        order_market_value(symbol=data.symbol, value=100) 
```

## **使用较长时间框架的趋势指标来衡量长期趋势方向**

长期趋势是由许多较小的趋势组成的，这就是为什么理解“更大的图景”对策略的执行很重要。多时间框架策略可以利用长期趋势来避免大规模提款，实现更高的胜率。

在熊市，简单的均线交叉策略的风险调整交易结果很差。这是因为它在寻找可能不存在的趋势。或者，即使有，也不会持续很久。

```py
###############################################################
# Multi symbol 1h moving average cross over strategy
###############################################################
SYMBOLS=["BTCUSDT", "ETHUSDT"]

def initialize(state):
    pass

@schedule(interval="1h", symbol=SYMBOLS, window_size=200)
def handler_1h(state, dataMap):
    for symbol, data in dataMap.items():
        if data is None: 
            continue

        ema1 = data.ema(10)
        ema2 = data.ema(50)

        port = query_portfolio()
        pos = query_open_position_by_symbol(symbol=data.symbol, include_dust=False)

        if pos is None :
            if ema1[-1] > ema2[-1]:
						    trade_size = port.portfolio_value * Decimal(.995) / len(SYMBOLS)
                trade_size = min(query_balance_free(data.quoted), trade_size)
                order_market_value(symbol=data.symbol, value=trade_size)
        else:
            if ema1[-1] < ema2[-1]:
                close_position(data.symbol) 
```

这个策略在上涨趋势中表现很好，但是当市场下跌时，它真的很挣扎。这可以从 PnL 的表现和最大水位下降看出。

![](img/29f8670a331ebf86eef5cef7f1d51984.png)![EMA crossover strategy, bear market, PnL](img/56c804ee3fab20e6f69f1f0cf79d6a24.png)



An EMA crossover strategy can struggle in a bearish market.



增加一个长期(1d) super_trend 指标，可以显著提高 bot 应对长期熊市的能力。

```py
###############################################################
# Multi symbol 1h moving average cross over strategy
# With 1d super trend indicator
###############################################################

SYMBOLS=["BTCUSDT", "ETHUSDT"]

def initialize(state):
    state.st_trend = {}

@schedule(interval="1d", symbol=SYMBOLS, window_size=200)
def handler_1d(state, dataMap):
    for symbol, data in dataMap.items():
        if data is None: 
            continue

        st = data.super_trend(24, 3.0)
        state.st_trend[symbol] = st["trend"][-1]

@schedule(interval="1h", symbol=SYMBOLS, window_size=200)
def handler_1h(state, dataMap):
    for symbol, data in dataMap.items():
        if data is None: 
            continue

        ema1 = data.ema(10)
        ema2 = data.ema(50)

        port = query_portfolio()
        pos = query_open_position_by_symbol(symbol=data.symbol, include_dust=False)
        if state.st_trend.get(data.symbol,0) > 0 :
            if pos is None :
                if ema1[-1] > ema2[-1]:
								    trade_size = port.portfolio_value * Decimal(.995) / len(SYMBOLS)
	                  trade_size = min(query_balance_free(data.quoted), trade_size)
	                  order_market_value(symbol=data.symbol, value=trade_size)
            else:
                if ema1[-1] < ema2[-1]:
                    close_position(data.symbol)
        else:
                close_position(data.symbol) 
```

上面的 bot 在 1 天处理程序中增加了 super_trend 指标的使用，以指示 1 小时处理程序的长期趋势是什么。这在回溯测试中的作用是，当长期趋势为熊市时，显著减少下降。以 PnL/提款衡量的简单策略的绩效从 0.1727 提高到 4.22，这是一个显著的提高。

![](img/29f8670a331ebf86eef5cef7f1d51984.png)![](img/97f7dc5e1646e08d974ce69f3a303497.png)





## **使用风险价值(VaR)衡量头寸，使风险与市场波动成比例**

损失通常发生在市场崩溃期间，此时波动性明显高于正常水平。使用价格协方差和当前头寸监控策略的风险并限制策略的风险是有益的，不是基于账户余额，而是基于由 VaR 估计的策略的总风险。

```py
import numpy as np

##############################################################
# User configs

SYMBOLS = ["XMRUSDT", "ETHUSDT", "DOGEUSDT"] 

ATR_PERIODS = 24
RISK_TARGET = 25.0    # risk target for strategy

##############################################################

def clamp(x, lo, hi):
    return min(hi, max(x, lo))

'''
Compute the risk of the portfolio
'''
def compute_portfolio_risk(dataMap, plot_risk=True) :
    bars_in_day = 24

    # compute 
    for symbol, data in dataMap.items() :
        bar_len_secs = (data.times[-1] - data.times[-2]) / 1000
        bars_in_day = (24*60*60) / bar_len_secs
        break

    risk = 0
    try:
        # compute portfolio risk 
        returns, vols = [], []
        positions = []
        for symbol, data in dataMap.items() :
            if data is None :
                continue 
            log_returns = np.diff(np.log(data.select("close"))[1:])
            returns.append(log_returns)
            vols.append(data.atr(ATR_PERIODS)[-1]/float(data.close.last))
            pos = query_open_position_by_symbol(symbol=data.symbol, include_dust=False)
            positions.append(float(0 if pos is None else pos.position_value))

            # plot position
            plot_line("pos", 0.0 if pos is None else float(pos.position_value), symbol=data.symbol)

        # create a covariance matrix using ATR
        corr = np.corrcoef(returns)
        covar = np.matmul(corr, np.diag(np.square(vols)))

        # compute portfolio risk
        risk = np.sqrt(np.matmul(np.matmul(np.transpose(positions), covar), positions) * bars_in_day)

        if plot_risk :
            for symbol, data in dataMap.items() :
                plot_line("risk", risk, symbol)
    except Exception as e:
        log(f"error computing risk {e}", severity=3)

    return risk

def initialize(state):
    pass

@schedule(interval="1h", symbol=SYMBOLS, window_size = 200)
def handler(state, dataMap):
    # calculate and plot portfolio risk
    portfolio_risk = compute_portfolio_risk(dataMap, plot_risk=True)

    for symbol, data in dataMap.items() :
        #pos = query_open_position_by_symbol(symbol, include_dust=False)

        pweight = float(query_position_weight(symbol=data.symbol))
        if state.run == 0 :
            starget = 0.5 / len(SYMBOLS)
        else:
            starget = pweight * (RISK_TARGET / portfolio_risk)

        # check if the difference between 
        if abs(starget - pweight) * float(query_portfolio_value()) > 25.0:
            order_market_target(symbol, starget) 
```

随着投资组合风险的增加，头寸会减少，以将总风险保持在一定范围内。

![](img/2f9b270b66d65d790ddefd4f6c501332.png)![](img/4ce58557a4391296ccdb2bf24f31b7d2.png)





重要的函数是 compute_portfolio_risk，它给出了第二天投资组合的预期波动率。

```py
portfolio_risk = compute_portfolio_risk(dataMap, plot_risk=True) 
```

使用这种投资组合风险，可以在风险超过限制时做出决策。在 bot 的情况下，我们的目标是相对于风险和我们的目标风险的比率的头寸规模。

```py
starget = pweight * (RISK_TARGET / portfolio_risk) 
```

这自然会在波动性低时增加头寸，在波动性高时减少头寸。

## **资产选择**

策略优化过程的另一个重要部分是资产选择，这也与市场机制密切相关，因为一些资产倾向于一种市场机制而不是另一种市场机制。像 [SHIB](https://shibatoken.com/) 和[总督](https://dogecoin.com/)这样的硬币可以有很高的波动性，并且在历史上有很强的趋势。像 [PAXG](https://paxos.com/paxgold/) 这样的硬币具有非常不同的动态，往往更加稳定，噪音更高，波动性更低。 [BTC](https://bitcoin.org/en/) 和 [ETH](https://ethereum.org/en/) 在市场上拥有大部分的交易量，并趋于趋势。

虽然每个硬币都有自己的动态，但相似的硬币也往往有相似的动态。用一个硬币赚钱的策略也可能通过交易相关性高的硬币赚钱。当查看跨许多资产的相关性时，相关性倾向于分组成簇。一个特定的交易策略可能对一个集群中的所有资产具有相似的交易性能。然后，您可以使用这些集群来形成一篮子资产。

有些策略，比如趋势跟踪策略，更喜欢倾向于趋势的资产。均值回归策略，如布林线策略，更喜欢倾向于波动的资产。为给定的策略选择合适的资产非常重要。

## **最终想法**

交易策略可以被认为是工具箱中的工具，每一个(或它们的组合)都是专门为从特定的市场机制中获利而设计的。每个市场体系都有描述价格动态本质的特征。作为一个交易者，最重要的是利用你所掌握的交易工具，并把它们和合适的市场机制结合起来，以达到最好的结果。这不是一劳永逸的方法，而是一个持续的过程，需要定期监控，并根据您的目标以及当前和预期的市场条件进行微调。