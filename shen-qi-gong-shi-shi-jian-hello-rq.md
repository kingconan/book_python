## 神奇公式

```
1 Establish a minimum market capitalization (usually greater than $50 million).
2 Exclude utility and financial stocks.
3 Exclude foreign companies (American Depositary Receipts).
4 Determine company's earnings yield = EBIT / enterprise value.
5 Determine company's return on capital = EBIT / (net fixed assets + working capital).
6 Rank all companies above chosen market capitalization by highest earnings yield and highest return on capital (ranked as percentages).
7 Invest in 20–30 highest ranked companies, accumulating 2–3 positions per month over a 12-month period.
8 Re-balance portfolio once per year, selling losers one week before the year-mark and winners one week after the year mark.
9 Continue over a long-term (5–10+ year) period.
```

```
1 排除掉公共事业和金融板块的股票。（为什么呢？这个可能需要从作者的那本书名很俗的书中寻找）
2 排除掉境外的公司（A股基本没有）
3 选取公司的息税前盈余/企业价值高的 = EBIT / Enterprise Value
    #便宜的股票是指税前盈余/企业价值（EBIT/EV）高的股票。
4 选取公司的有形资本回报率（return on capital）= EBIT / (Net Fixed Assets + Working Capital) 
    #好的业务是指有形资本回报率高的公司
5 从所有市面上的股票中选取3和4最高的公司的股票
6 投资20-30个排名最高的股票，然后每个月持续加入2-3个持仓直到超过12个月
7 每年进行重新调整投资组合，年前剔除掉当年投资比较失败的，年后剔除掉当年投资比较成功的
8 长期继续以上的操作（5-10年+）
```

#### 股票板块信息

```py
#获得属于某个板块的所有股票列表
#parameter : code表示板块名称或者板块代码，eg:Energy，能源或者sector_code.Energy，
#return : 股票order?book_id列表
sector(code)
```

#### 涉及的财务指标

```
EBIT息税前利润 : 息税前利润，是扣除利息、所得税之前的利润。
ev : 企业价值 = 股权价值 + 带息债务 （含货币资金）
ev_2 : 企业价值 = 股权价值 + 带息债务 - 货币资金
return_on_invested_capital : 投入资本回报率ROIC，
  是指投出和/或使用资金与相关回报（回报通常表现为获取的利息和/或分得利润）之比例。用于衡量投出资金的使用效果。
```

#### 如何查询财务数据

```py
get_fundamentals(query, entry_date=None, interval='1d', report_quarter=False)
```

#### code

```py
def init(context):
    context.exclude_stocks = sector('utilities') + sector('financial')
    logger.info(context.exclude_stocks)

def magic_stocks(context, bar_dict):
    #target 3，ev_to_ebit 正好是目标3的倒数，故升序
    t3 = get_fundamentals(
        query(fundamentals.eod_derivative_indicator.ev_to_ebit)
            .order_by(fundamentals.eod_derivative_indicator.ev_to_ebit.asc())
            .limit(300)
    )
    #target 4
    t4 = get_fundamentals(
        query(fundamentals.financial_indicator.return_on_invested_capital)
            .order_by(fundamentals.financial_indicator.return_on_invested_capital.desc())
            .limit(300)
    )
    #target 5
    t5 = [stock for stock in t3 if stock in t4]
    #target 2
    t2 = [stock for stock in t5 if stock not in context.exclude_stocks]
    #target 6
    t6 = t2[0:30]
    context.magic_stocks = t6

    logger.info(context.magic_stocks)

    #2016-06-24 INFO  
    #[
    # '000550.XSHE', '002419.XSHE', '600741.XSHG', '000651.XSHE', '000625.XSHE', 
    # '000626.XSHE', '600694.XSHG', '600583.XSHG', '600690.XSHG', '000581.XSHE', 
    # '600600.XSHG', '000828.XSHE', '600004.XSHG', '600177.XSHG', '601633.XSHG', 
    # '600826.XSHG', '600100.XSHG', '000333.XSHE', '600299.XSHG', '600129.XSHG', 
    # '600897.XSHG', '600688.XSHG', '000418.XSHE', '601238.XSHG', '601877.XSHG', 
    # '002561.XSHE', '600060.XSHG', '000858.XSHE', '000895.XSHE', '603188.XSHG'
    #]
```

#### 参考

```
https://www.ricequant.com/api/python/chn#scheduler-time
https://www.ricequant.com/fundamentals#ana_stk_fin_idx
http://docs.sqlalchemy.org/en/rel_1_0/orm/tutorial.html#querying
```



