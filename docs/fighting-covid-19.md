# 在新冠肺炎疫情期间，金融市场会发生什么？定量分析师的分析

> 原文：<https://www.trality.com/blog/fighting-covid-19/>

新冠肺炎几乎影响着我们生活的方方面面。恐惧和不确定性正在动摇全球金融市场和加密货币。自我隔离正在减缓我们的生活和社会交往。然而，从积极的方面来看，我们有更多的时间来消费各种信息。因此，我们想借此机会为您提供最近事件的快速时间序列分析。

在下面的博客文章中，我们展示了如何使用 Python 从定量的角度获得当前市场环境的概况。我们从 Kaggle、加密货币市场和传统资产类别数据中探索 COVID 数据集。这绝不意味着是一篇科学论文，而是一个概念和思想的说明。

更准确地说，我们使用 [Kaggle](https://www.kaggle.com/sudalairajkumar/novel-corona-virus-2019-dataset/data#covid_19_data.csv) 来可视化病毒在各国的爆发情况。下一步，我们来看看来自[币安](https://www.binance.com/en)的加密货币数据，以检验在极度波动和危机时期常见的一些典型事实。然后，我们将这些影响与传统资产类别的数据进行比较。最后，我们对加密货币市场进行了相关性分析，表明“传染”主要是由逃向安全稳定的硬币所驱动的。

## 可视化**COVID 数据集**

我们使用来自 [Kaggle](https://www.kaggle.com/sudalairajkumar/novel-corona-virus-2019-dataset/data#covid_19_data.csv) 的每日数据集来可视化已确认的电晕案例的数量及其增长率。我们重点关注按字母顺序排列的以下国家/地区的数据点:

*   奥地利
*   法国
*   德国
*   意大利
*   Mainland China
*   韩国
*   西班牙
*   英国
*   美国

这种国家/地区的选择(非常主观地)是基于疾病爆发的严重性和/或对世界经济的重要性。

在观察了确诊病例的数量和增长率后，我们还计算了每个观察日期的官方感染人数(确诊-康复病例)，以了解在什么水平上发生了强烈的市场调整([黑色星期一和黑色星期四](https://en.wikipedia.org/wiki/2020_stock_market_crash#Black_Monday_(2020)))。

在开始之前，我们加载数据和时间序列分析常用的包。我们目前在自己的[城市编辑器](https://www.trality.com/creator/code-editor)中支持`numpy`和`pandas`。我们使用`matplotib`和`seaborn`来获得更高级的图表功能。

```py
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
import seaborn as sns
```

现在，我们准备探索 COVID 数据集。

```py
dataset = pd.read_csv("https://static.trality.com/blog/covid19/covid_19_data.csv")
columns = ['SNo', 'ObservationDate', 'State', 'Country',
           'Last Update', 'Confirmed', 'Deaths', 'Recovered']
dataset.columns = columns
dataset.head()
```

![](img/029cb515ff711444a7ee92ab05097381.png)![](img/cd94f56804e5da2c72960138b8368025.png)





如 Kaggle 所述，相关列有以下解释:

*   **Sno** -序列号
*   **观察日期** -观察日期，单位为年/月/日
*   **省/州** -观察的省或州(缺少时可以为空)
*   **国家/地区** -观察国
*   **Last Update**-UTC 中给定省份或国家更新行的时间。(未标准化，因此请在使用前清洁)
*   **确诊** -截止当日累计确诊病例数
*   **死亡人数** -截至该日期的累计死亡人数
*   **已恢复** -到该日期为止的累计已恢复病例数

如 Kaggle 的描述中所述，实际病例数(确诊、痊愈和死亡)可能落后于相关观察日期。出于演示的目的，我们坚持观察日期。

为了可视化病毒的爆发，我们首先转换数据以获得确诊和痊愈病例以及死亡的时间序列。为此，我们编写了一个简单的助手函数，它返回字典中所选国家的所有时间序列。

```py
def get_corona_timeseries(dataset,selcountries=None,
						  group="Country"):

    tsnames = ["Confirmed","Deaths","Recovered"]

    if not selcountries is None:
        selidx = [True if country in selcountries else False 
                  for country in dataset["Country"]]
        dataset = dataset.loc[selidx]

    tsdata = dataset.groupby(["ObservationDate",group])
    results = {}

    for name in tsnames:
        tmpres = {}
        series = tsdata[name].sum().reset_index()
        series = series.pivot(index="ObservationDate",
                              columns=group,values=name)
        series.index = pd.DatetimeIndex(series.index)
        tmpres["series"] = series
        growth = (1+series.pct_change().dropna()).cumprod()
        growth.iloc[0,0] = 1
        tmpres["growth"] = growth

        results[name] = tmpres

    return results
```

为了坚持[干](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)原则，我们使用另一个效用函数来绘图，

```py
def corona_plotter(data,ylabel,title,
				   xlabel="Observation Date",
                   figsize=(10,4)):
    fig, ax = plt.subplots(figsize=figsize)
    ax.set(xlabel=xlabel,ylabel=ylabel,title=title)
    ax.xaxis.set_major_locator(mdates.WeekdayLocator())
    ax.xaxis.set_major_formatter(mdates.DateFormatter('%b %d'))
    data.plot(ax=ax)
    plt.legend(loc='center left', bbox_to_anchor=(1.0, 0.5))

selcountries = ["Germany","France","US","UK",
                "Mainland China","Italy","Austria","Spain",
                "South Korea"]
results = get_corona_timeseries(dataset,selcountries=selcountries)
```

我们首先来看看在我们选定的国家中，每个观测日确诊的电晕病例数。

```py
confirmed = results["Confirmed"]["series"]
corona_plotter(confirmed,ylabel="Confirmed Cases",
			   title="Total Number confirmed cases") 
```

![](img/1fff71cb905a92e9bbdd5b444696d5f4.png)![](img/403af1b9ddb45a59dcbb1ad121dfc7b5.png)





显然，中国确诊病例最多。然而，就增长率而言，欧洲的情况要糟糕得多。

```py
confirmedGrowth = results["Confirmed"]["growth"]
corona_plotter(confirmedGrowth,
				ylabel="Confirmed Growth",
                title="Growth rates of Confirmed Cases")
```

![](img/8d33dce8e696608fb8d47f070efe6c7f.png)![](img/73b7d937852e5cc9d064ce5f2a1c905d.png)





最后，我们计算每个观察日期的“官方”感染人数(确认-恢复)。尽管实际感染人数远高于官方报告，但看到市场在多高的水平才开始抛售，仍然很有趣。由于中国的感染人数已经在减少，我们在这里排除了他们。

```py
infected = (results["Confirmed"]["series"] 
			- results["Recovered"]["series"])

# removing china
selcols = [col for col in infected.columns 
			if col!="Mainland China"]

totalinfected = infected[selcols].sum(axis=1)
# transforming dates - quick fix for annotation
totalinfected.index = [pd.Timestamp(str(date.date())) 
					   for date in totalinfected.index] 

title = """Total of confirmed Infected 
			per Observation Date 
			without China (excluding Recovered)"""

corona_plotter(totalinfected,ylabel="Infected people",
               title=title) 
```

![](img/c42d9326954031844a3ae48cd1e16385.png)![](img/42ce53c538c2d36300721339dadccf18.png)





在黑色星期一，我们选定国家的官方感染人数达到 20，000 多人。在黑色星期四，这个数字增加到了 27000 多。截至 3 月 22 日，人数超过 164，000 人。

从数学建模的角度来看，这些严重的传染效应也已经被彻底研究过了。传染病建模最简单的方法之一是 [SIR 模型](https://www.maa.org/press/periodicals/loci/joma/the-sir-model-for-spread-of-disease-the-differential-equation-model)。

## 探索加密货币数据集

该数据集取自[币安](https://www.binance.com/en)，包含每日价格，范围从 2019 年初到现在(2020 年 3 月 20 日)。我们考虑了 14 种主要货币，分别是 USDT、BTC 和欧元。由于黑色星期一和星期四的大部分交易影响了 USDT 对，包括稳定的硬币进入分析似乎是自然的。

每个符号的数据相互叠加并包含在`alldata`中。我们显示了前 5 行来查看它的结构。

```py
alldata = pd.read_csv("https://static.trality.com/blog/covid19/crypto_data_1d.csv")
alldata.head()
```

![](img/0bd7d5321ada4dc1b06cb1fd0d601930.png)![](img/cb37405ffb2474726a8eceba58804b38.png)





在这里，我们可以看到不同符号的开盘价、最高价、最低价、收盘价、成交量和交易数据。`closetime`表示每日间隔的结束时间(UTC+2)。

现在我们已经看到了我们的堆叠数据集，我们希望隔离所有货币对的价格、交易量和交易。为了简化，我们将结果存储在下面的`tsdata`字典中:

```py
tsdata = {}
for field in ["close","volume","trades"]:
    data = alldata.pivot(values=field,index="closetime",
    					columns="symbol").dropna()
    data = data.astype("float64")
    data.index = pd.DatetimeIndex(data.index)
    tsdata[field] = data 
```

下一步，我们将关注 2020 年 3 月货币对的价格演变。

### **严重电晕对加密价格的影响**

为了使冠状病毒对不同货币价格的影响具有可比性，我们从 3 月初开始对货币对的时间序列进行归一化，并对符号进行平均。

```py
returns = tsdata["close"].pct_change().dropna()
covidret = returns["2020-03"]
normdata = (1+covidret).cumprod()
normdata.iloc[0] = 1
meanhist = normdata.mean(axis=1)
```

我们现在可以绘制这个时间序列，并注释黑色星期一和星期四。

```py
fig, ax = plt.subplots(figsize=(10, 4))

blackmonday = normdata["2020-03-09"].index[0]
blackthursday = normdata["2020-03-13":].index[0]
price1 = normdata["2020-03-09":].iloc[0,0]
price2 = normdata["2020-03-13":].iloc[0,0]

meanhist.plot(ax=ax)

title = 'Average Markt Evolution March 2020 - normalized'
ax.set(title=title,
       ylabel='Normalized Values',
       xlabel="Date")

ax.annotate('Black Monday',
            xy=(blackmonday, price1),
            xycoords='data',
            xytext=(30,30),
            textcoords='offset points',
            arrowprops=dict(headwidth=10, 
            width=3, color='#363d46',
            connectionstyle="angle3,angleA=0,angleB=-90"),
            fontsize=12)

ax.annotate('Black Thursday',
            xy=(blackthursday, price2),
            xycoords='data',
            xytext=(10,30),
            textcoords='offset points',
            arrowprops=dict(headwidth=10, 
            width=3, color='#363d46',
            connectionstyle="angle3,angleA=0,angleB=-90"),
            fontsize=12) 
```

![](img/624176168553a47122ff7449fcb62efe.png)![](img/fe57c016ad6a33585a0fa3549cf6a923.png)





正如我们所看到的，加密市场的第一次冲击开始于 3 月 9 日(黑色星期一)。加密市场的真正传染始于 3 月 13 日的黑色星期四，一些货币对下跌超过 50%。自 3 月初以来，市场平均市值已缩水 30%以上。

### 抛售期间交易量激增

恐惧和不确定性对交易量也有很大影响，通常与价格变化成反比。为了标准化体积数据，我们使用了[最小-最大](https://en.wikipedia.org/wiki/Feature_scaling)定标器。这样就可以避免度量单位不同和级别依赖的问题。我们再次取体积的平均值。正如我们所见，交易量的激增与平均价格的下降同时发生。

```py
volume = tsdata["volume"]
normvol = (volume-volume.min())/(volume.max()-volume.min())
covidvolume = normvol["2020-03":]

covidvolume.mean(axis=1).plot(figsize=(10,4),
							title="Average Scaled Volume")
```

![](img/53192332bed5b53ed06e305060d74290.png)![](img/a35024993cbf408c1f64eb1b84152df1.png)





### 萧条时期市场的剧烈波动

金融回报的一个特点是，在市场低迷时期，极端的市场波动往往会伴随着极端的市场波动。这种现象被称为[波动聚集](https://en.wikipedia.org/wiki/Volatility_clustering)。它的后果是深远的，因为金融中一些常用的假设被打破了——最明显的是常态假设。关于详细的解释，读者可以参考这篇关于[资产回报](https://orfe.princeton.edu/~jqfan/fan/FinEcon/chap1.pdf)的论文。

通常我们可以通过绘制绝对收益来检验波动聚集。高绝对回报集中在市场紧张时期。出现在集群中。

```py
returns.abs().plot(figsize=(10,4),
			title="Absolute Returns Currencies")
plt.legend(loc='center left', bbox_to_anchor=(1.0, 0.5))
```

![](img/5e726169e921e2fdb83281890c5d77ba.png)![](img/e28f1b3a7dcfef69e18852e44dad1ceb.png)





尽管历史还不够长，但我们看到了集群的初步迹象，这种情况似乎很可能会持续一段时间。

## **探索传统资产类别**

在这一节中，我们来看看传统资产类别的价格时间序列。我们的数据集由商品、指数和货币的各种时间序列组成。

*   ****商品**** :黄金、石油、
*   ****指数**** : S & P 指数(美国)、30y 国债收益率(美国)、恒生指数(中国)、SX5E(欧洲)、富时(英国)
*   ****货币**** :欧元美元货币，GBPUSD 货币

我们的数据集包含来自 YahooFinance 的每日价格数据。

![](img/084ef99cfa758b7704cddd9664deaeef.png)![](img/998a7378822f714d294aa698d6c64bb5.png)





YahooFinance 还包括开盘价、最高价、最低价、收盘价和成交量数据。字段 ticker 表示相应的商品、指数或货币。由于这些时间序列不代表直接交易的工具，我们忽略了交易量数据。

### **商品和股票市场的强劲表现**

与之前的加密货币数据一样，我们关注的是对传统资产类别市场的平均价格影响。

```py
alldata2 =pd.read_csv("https://static.trality.com/blog/covid19/traditional_assets_daily.csv
")

tsdata2 = {}
for field in ["close","volume"]:
    data=alldata2.pivot(values=field,
    					index="date",
                        columns="ticker").dropna()
    data = data.astype("float64")
    data.index = pd.DatetimeIndex(data.index)
    tsdata2[field] = data
```

我们再次将传统资产的时间序列正常化，并强调 2020 年 3 月的发展。

```py
returns2 = tsdata2["close"].pct_change().dropna()
covidret2 = returns2["2020":]
normdata2 = (1+covidret2).cumprod()
normdata2.iloc[0] = 1

fig, ax = plt.subplots(figsize=(10, 4))

oilprice = normdata2.loc[blackmonday.date(),"OIL"]

# highlight march period
ax.axvspan(normdata2["2020-03"].index[0], 
			normdata2["2020-03"].index[-1], 
			color='grey', alpha=0.5)
normdata2.plot(ax=ax)

ax.set(title='Normalized timeseries Asset Classes',
       ylabel='Normalized Values',xlabel="Date")

ax.annotate('Corona/Oil War',
            xy=(blackmonday, oilprice),
            xycoords='data',
            xytext=(-10,-30),
            textcoords='offset points',
            arrowprops=dict(headwidth=10, 
            width=3, color='#363d46',
            connectionstyle="angle3,angleA=0,angleB=-90"),
            fontsize=12)

plt.legend(loc='center left', bbox_to_anchor=(1.0, 0.5))
```

![](img/e8b2f94686f693307b0c32945655a55c.png)![](img/87a191ae21182f81260259723091825d.png)





与我们的加密货币数据集相反，它只代表一个市场，我们在这里看到非常多样化的市场。然而，截至 3 月份，大多数资产类别都在朝着同一个方向发展，当然，债券收益率除外——它们与价格成反比关系。

### **剧烈的价格波动影响所有资产类别**

绘制传统资产类别的绝对收益图也显示了波动性的聚集效应。

```py
returns2["2019":].abs().plot(title="Absolute Returns")
plt.legend(loc='center left', bbox_to_anchor=(1.0, 0.5))
```

![](img/8d194394f80e27eea6bfe268da66ea98.png)![](img/5143ccecdabdf87097c47872f19b4df7.png)





对病毒传播及其后果的担忧似乎以类似的方式感染了传统和加密货币市场。至少乍看起来，我们在价格和波动性上发现了类似的反应。

## 加密货币的相关性和溢出效应

有很多关于加密货币高相关性的报道。2019 年[币安报告](https://research.binance.com/analysis/annual-crypto-correlations-2019)指出，加密市场的中值相关性继续保持高位。大众媒体似乎也很担心。[例如，CoinTelegraph](https://cointelegraph.com/news/truth-about-crypto-price-correlation-how-closely-does-eth-follow-btc) 谈到了 BTC 对 ETH 的影响，反之亦然。除了因果关系和相关性之间的误解，它提出了 BTC 和 ETH 的关系两种对立的观点。估计时间序列中的相关性带来了很多挑战(见 [UCL 研究报告](http://www.cs.ucl.ac.uk/fileadmin/UCL-CS/research/Research_Notes/RN_11_01.pdf))。

电晕危机对 crypto 的影响导致了对安全的逃离。因此，我们期望在稳定的硬币对和 BTC 对之间看到不同的平均相关性。为了得到指示，我们将数据分成两组。一套包含所有基础货币稳定的货币对(USDT、欧元)，另一套包含 BTC 货币对。

为了缓解估计时间序列相关性的问题，可以监控滚动相关性，并在滚动期的每个窗口中取消趋势回报(参见 Crypto 中的 [Contagion)。出于我们的目的，我们将简单地归一化而不是去趋势化各个窗口中的所有返回时间序列。为此，我们创建了一个小助手函数。](https://www.researchgate.net/publication/334363449_Contagion_Effect_in_Cryptocurrency_Market)

```py
def avg_rolling_correlation(data,window=10):

    if not isinstance(data,pd.DataFrame):
        raise TypeError("only dataframes supported!")

    timestamps = data.index
    result = pd.Series(index=timestamps)

    windows = zip(timestamps[:-window],timestamps[window:])

    for start,end in windows:
        tmpdata = data.loc[start:end]
        normdata = (tmpdata-tmpdata.mean())/tmpdata.std()
        meancorr = normdata.corr().values.mean()
        result[end] = meancorr

    return result 
```

接下来，我们计算两组的滚动相关性，并绘制结果。

```py
selcols = [col for col in returns.columns if (col[-4:]=="USDT" or col[-3:]=="EUR")]
stableCorr = avg_rolling_correlation(returns[selcols]).dropna()
stableCorr.name="stable coins"

selcols2 = [col for col in returns.columns if col not in selcol]
otherCorr = avg_rolling_correlation(returns[selcols2]).dropna()
otherCorr.name = "BTC"

corrdata = stableCorr.to_frame()
corrdata = corrdata.merge(otherCorr
                            ,on="closetime",how="inner")

print(selcols2)
ax = corrdata.plot()

ax.set_title("Average correlations in stable coin vs BTC pairs")
ax.set_ylabel("Average Correlation")
```

![](img/7efa668cc0f53e93c3abea5bf22bbdf0.png)![](img/6055415697e21b350221bd3853009a82.png)





综上所述，市场中极高的相关性似乎主要是由投资稳定的安全货币所驱动的。对流动性的担忧和其他担忧可能加剧了这些状况。所有其他货币对的相关性较低，可能会更快恢复到正常水平。

从经验上看，滚动相关性往往是提前反映市场情绪的良好指标。由于它们的均值回复性质，它们在交易中有广泛的应用。

## 定量分析以避免偏差

危机情况通常会导致典型的市场模式，这在传统资产类别和加密货币中都可以观察到。分析和理解这些模式所需的大部分信息都是公开的，很容易检索，这使得定量分析成为理解这些信息的有力工具。上述分析的关键要点是:

*   价格和交易量往往是负相关的。高交易量通常伴随着价格下跌。每个人都想同时出去。
*   恐慌会引起更大的恐慌。高价格变化往往伴随着高价格变化。就波动性而言，我们可以观察到聚集效应。
*   避险行为已经将稳定硬币对的平均相关性推至极高水平。BTC 对的相关性要低得多，而且似乎稳定得更快。
*   Python 很酷，我们应该更多地使用它

我们希望你喜欢这个。如果您有任何想法、意见或建议，请告诉我们。请在`[[email protected]](/cdn-cgi/l/email-protection)`下给我们发电子邮件。