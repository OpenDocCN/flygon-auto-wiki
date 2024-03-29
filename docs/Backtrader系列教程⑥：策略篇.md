<!--yml
category: 交易
date: 2023-09-17 19:49:34
-->

# Backtrader系列教程⑥:策略篇

> 来源：[https://blog.csdn.net/qq_41578115/article/details/122530482](https://blog.csdn.net/qq_41578115/article/details/122530482)

今天的《策略篇》先会对 Strategy 常规策略实现操作做一个汇总，然后再介绍一种更为简单的策略实现方式——信号策略；还会重点介绍策略收益评价指标的生成方式；最后会介绍策略参数优化功能。

**通过 Strategy 类开发策略**

从《Backtrader 来了～》到现在，相信大家对 Backtrader 中的 Strategy 策略类应该不再陌生了，知道策略逻辑都写在 Strategy 类里，还知道 Strategy 类里有__init__() 、next() 、notify_order()、notify_trade() 等方法，有各式各样的交易函数，有各式各样的查询函数，下面就将这些内容做一个汇总：

```
import backtrader as bt # 导入 Backtrader
# 创建策略
class MyStrategy(bt.Strategy):
# 初始化策略参数
params = (
(...,...), # 最后一个“,”最好别删！
)
# 日志打印：参考的官方文档
def log(self, txt, dt=None):
'''构建策略打印日志的函数：可用于打印订单记录或交易记录等'''
dt = dt or self.datas[0].datetime.date(0)
print('%s, %s' % (dt.isoformat(), txt))
# 初始化函数
def __init__(self):
'''初始化属性、计算指标等'''
# 指标计算可参考《backtrader指标篇》
self.add_timer() # 添加定时器
pass
# 整个回测周期上，不同时间段对应的函数
def start(self):
'''在回测开始之前调用,对应第0根bar'''
# 回测开始之前的有关处理逻辑可以写在这里
# 默认调用空的 start() 函数，用于启动回测
pass
def prenext(self):
'''策略准备阶段,对应第1根bar-第 min_period-1 根bar'''
# 该函数主要用于等待指标计算，指标计算完成前都会默认调用prenext()空函数
# min_period 就是 __init__ 中计算完成所有指标的第1个值所需的最小时间段
pass
def nextstart(self):
'''策略正常运行的第一个时点，对应第 min_period 根bar'''
# 只有在 __init__ 中所有指标都有值可用的情况下，才会开始运行策略
# nextstart()只运行一次，主要用于告知后面可以开始启动 next() 了
# nextstart()的默认实现是简单地调用next(),所以next中的策略逻辑从第 min_period根bar就已经开始执行
pass
def next(self):
'''策略正常运行阶段，对应第min_period+1根bar-最后一根bar'''
# 主要的策略逻辑都是写在该函数下
# 进入该阶段后，会依次在每个bar上循环运行next函数
# 查询函数
print('当前持仓量', self.getposition(self.data).size)
print('当前持仓成本', self.getposition(self.data).price)
# self.getpositionbyname(name=None, broker=None)
print('数据集名称列表',getdatanames())
data = getdatabyname(name) # 根据名称返回数据集
# 常规下单函数
self.order = self.buy( ...) # 买入、做多 long
self.order = self.sell(...) # 卖出、做空 short
self.order = self.close(...) # 平仓 cover
self.cancel(order) # 取消订单
# 目标下单函数
# 按目标数量下单
self.order = self.order_target_size(target=size)
# 按目标金额下单
self.order = self.order_target_value(target=value)
# 按目标百分比下单
self.order = self.order_target_percent(target=percent)
# 订单组合
brackets = self.buy_bracket()
brackets = self.sell_bracket()
pass
def stop(self):
'''策略结束，对应最后一根bar'''
# 告知系统回测已完成，可以进行策略重置和回测结果整理了
pass
# 打印回测日志
def notify_order(self, order):
'''通知订单信息'''
pass
def notify_trade(self, trade):
'''通知交易信息'''
pass
def notify_cashvalue(self, cash, value):
'''通知当前资金和总资产'''
pass
def notify_fund(self, cash, value, fundvalue, shares):
'''返回当前资金、总资产、基金价值、基金份额'''
pass
def notify_store(self, msg, *args, **kwargs):
'''返回供应商发出的信息通知'''
pass
def notify_data(self, data, status, *args, **kwargs):
'''返回数据相关的通知'''
pass
def notify_timer(self, timer, when, *args, **kwargs)：
'''返回定时器的通知'''
# 定时器可以通过函数add_time()添加
pass
# 各式各样的交易函数和查询函数：请查看《交易篇（上）》和《交易篇（下）》
......
# 将策略添加给大脑
cerebro.addstrategy(MyStrategy)
......
```

**基于交易信号直接生成策略**

除了在 Strategy 类中编写策略外，追求 “极简” 的 Backtrader 还给大家提供了一种更为简单的策略生成方式，这种方式不需要定义 Strategy 类，更不需要调用交易函数，只需计算信号 signal 指标，然后将其 add_signal 给大脑 Cerebro 即可，Cerebro 会自动将信号 signal 指标转换为交易指令，通常可以将这类策略称为信号策略 SignalStrategy 。下面以官方文档中的例子介绍信号策略生成方式：

 **add_signal(signal type, signal class, arg) 中的参数说明：**

    第 1 个参数：信号类型，分为 2 大类，共计 5 种信号类型：

**开仓类：**

*   step1：自定义交易信号，交易信号和一般的指标相比的区别只在于：交易信号指标在通过 add_signal 传递给大脑后，大脑会将其转换为策略，所以在自定义交易信号时直接按照 Indicator 指标定义方式来定义即可（具体可以参考之前的《指标篇》）。定义时需要声明信号 'signal' 线，信号指标也是赋值给 'signal' 线；
*   step2：按常规方式，实例化大脑 cerebro、加载数据、通过 add_signal 添加交易信号线 ；
*   备注1：信号策略每次下单的成交量取的是 Sizer 模块中的 FixedSize，默认成交 1 单位的标的，比如 1 股、1 张合约等；
*   备注2：生成的是市价单 Market，订单在被取消前一直都有效。

    ```
    import backtrader as bt
    # 自定义信号指标
    class MySignal(bt.Indicator):
    lines = ('signal',) # 声明 signal 线，交易信号放在 signal line 上
    params = (('period', 30),)
    def __init__(self):
    self.lines.signal = self.data - bt.indicators.SMA(period=self.p.period)
    # 实例化大脑
    cerebro = bt.Cerebro()
    # 加载数据
    data = bt.feeds.OneOfTheFeeds(dataname='mydataname')
    cerebro.adddata(data)
    # 添加交易信号
    cerebro.add_signal(bt.SIGNAL_LONGSHORT, MySignal, period=xxx)
    cerebro.run()
    ```

    支持添加多条交易信号:

    ```
    import backtrader as bt
    # 定义交易信号1
    class SMACloseSignal(bt.Indicator):
    lines = ('signal',)
    params = (('period', 30),)
    def __init__(self):
    self.lines.signal = self.data - bt.indicators.SMA(period=self.p.period)
    # 定义交易信号2
    class SMAExitSignal(bt.Indicator):
    lines = ('signal',)
    params = (('p1', 5), ('p2', 30),)
    def __init__(self):
    sma1 = bt.indicators.SMA(period=self.p.p1)
    sma2 = bt.indicators.SMA(period=self.p.p2)
    self.lines.signal = sma1 - sma2
    # 实例化大脑
    cerebro = bt.Cerebro()
    # 加载数据
    data = bt.feeds.OneOfTheFeeds(dataname='mydataname')
    cerebro.adddata(data)
    # 添加交易信号1
    cerebro.add_signal(bt.SIGNAL_LONG, MySignal, period=xxx)
    # 添加交易信号2
    cerebro.add_signal(bt.SIGNAL_LONGEXIT, SMAExitSignal, p1=xxx, p2=xxx)
    cerebro.run()
    ```

    **信号指标取值与多空信号对应关系：**

*   signal 指标取值大于0 → 对应多头 long 信号；
*   signal 指标取值小于0 → 对应空头 short 信号；
*   signal 指标取值等于0 → 不发指令； 
*   bt.SIGNAL_LONGSHORT：
*   多头信号和空头信号都会作为开仓信号；
*   对于多头信号，如果之前有空头仓位，会先对空仓进行平仓 close，再开多仓；
*   空头信号也类似，会在开空仓前对多仓进行平仓 close。
*   bt.SIGNAL_LONG：
*   多头信号用于做多，空头信号用于平仓 close；
*   如果系统中同时存在 LONGEXIT 信号类型，SIGNAL_LONG 中的空头信号将不起作用，将会使用 LONGEXIT 中的空头信号来平仓多头，如上面的多条交易信号的例子。
*   bt.SIGNAL_SHORT：
*   空头信号用于做空，多头信号用于平仓；
*   如果系统中同时存在 SHORTEXIT 信号类型，SIGNAL_SHORT 中的多头信号将不起作用，将会使用 SHORTEXIT 中的多头信号来平仓空头。

**平仓类：**

*   bt.SIGNAL_LONGEXIT：接收空头信号平仓多头；
*   bt.SIGNAL_SHORTEXIT：接收多头信号平仓空头；
*   上述 2 种信号类型主要用于确定平仓信号，在下达平仓指令时，优先级高于上面开仓类中的信号。
*   第 2 个参数：定义的信号指标类的名称，比如案例中的 SMACloseSignal 类 和 SMAExitSignal 类，直接传入类即可，不需要将类进行实例化；
*   第 3 个参数：对应信号指标类中的参数 params，直接通过 period=xxx 、p1=xxx, p2=xxx 形式修改参数取值。

**关于订单累计和订单并发：**

由于交易信号指标通常只是技术指标之间进行加减得到，在技术指标完全已知的情况下，很容易连续不断的生成交易信号，进而连续不断的生成订单，这样就容易出现如下 2 种情况：

*   积累 Accumulation：即使已经在市场上，信号也会产生新的订单，进而增加市场的头寸；
*   并发 Concurrency：新订单会并行着生成，而不是等待其他订单的执行完再后依次执行。

可通过如下 2 个函数来控制上述 2 种情况的发生：

```
cerebro.signal_accumulate(True)
cerebro.signal_concurrency(True)
# True 表示允许其发生， False 表示不允许其发生
```

**如何返回策略收益评价指标**

回测完成后，通常需要计算此次回测的各项收益评价指标，据此判断策略的好坏表现，在 Backtrader 中，有专门负责回测收益评价指标计算的模块 analyzers，大家可以将其称为“策略分析器”。关于 analyzers 支持内置的指标分析器的具体信息可以参考官方文档 Backtrader ~ Analyzers Reference 。分析器的使用主要分为 2 步：

*   第一步：通过 addanalyzer(ancls, _name, *args, **kwargs) 方法将分析器添加给大脑，ancls 对应内置的分析器类，后面是分析器各自支持的参数，添加的分析器类 ancls 在 cerebro running 区间会被实例化，并分配给 cerebro 中的每个策略，然后分析每个策略的表现，而不是所有策略整体的表现 ；
*   第二步：分别基于results = cerebro.run() 返回的各个对象 results[x] ，提取该对象 analyzers 属性下的各个分析器的计算结果，并通过 get_analysis() 来获取具体值。
*   说明：addanalyzer() 时，通常会通过 _name 参数对分析器进行命名，在第二步获取分析器结果就是通过_name 来提取的。

    ```
    ......
    # 添加分析指标
    # 返回年初至年末的年度收益率
    cerebro.addanalyzer(bt.analyzers.AnnualReturn, _name='_AnnualReturn')
    # 计算最大回撤相关指标
    cerebro.addanalyzer(bt.analyzers.DrawDown, _name='_DrawDown')
    # 计算年化收益：日度收益
    cerebro.addanalyzer(bt.analyzers.Returns, _name='_Returns', tann=252)
    # 计算年化夏普比率：日度收益
    cerebro.addanalyzer(bt.analyzers.SharpeRatio, _name='_SharpeRatio', timeframe=bt.TimeFrame.Days, annualize=True, riskfreerate=0) # 计算夏普比率
    cerebro.addanalyzer(bt.analyzers.SharpeRatio_A, _name='_SharpeRatio_A')
    # 返回收益率时序
    cerebro.addanalyzer(bt.analyzers.TimeReturn, _name='_TimeReturn')
    # 启动回测
    result = cerebro.run()
    # 提取结果
    print("--------------- AnnualReturn -----------------")
    print(result[0].analyzers._AnnualReturn.get_analysis())
    print("--------------- DrawDown -----------------")
    print(result[0].analyzers._DrawDown.get_analysis())
    print("--------------- Returns -----------------")
    print(result[0].analyzers._Returns.get_analysis())
    print("--------------- SharpeRatio -----------------")
    print(result[0].analyzers._SharpeRatio.get_analysis())
    print("--------------- SharpeRatio_A -----------------")
    print(result[0].analyzers._SharpeRatio_A.get_analysis())
    ......
    ```

    各个分析器的结果通常以 OrderedDict 字典的形式返回，如下所示，大家可以通过 keys 取需要的 values：

    ```
    AutoOrderedDict([('len', 56),
    ('drawdown', 8.085458202746946e-05),
    ('moneydown', 8.08547225035727),
    ('max',
    AutoOrderedDict([('len', 208),
    ('drawdown', 0.00015969111320873712),
    ('moneydown', 15.969112889841199)]))])
    # 常用指标提取
    analyzer = {}
    # 提取年化收益
    analyzer['年化收益率'] = result[0].analyzers._Returns.get_analysis()['rnorm']
    analyzer['年化收益率（%）'] = result[0].analyzers._Returns.get_analysis()['rnorm100']
    # 提取最大回撤
    analyzer['最大回撤（%）'] = result[0].analyzers._DrawDown.get_analysis()['max']['drawdown'] * (-1)
    # 提取夏普比率
    analyzer['年化夏普比率'] = result[0].analyzers._SharpeRatio_A.get_analysis()['sharperatio']
    # 日度收益率序列
    ret = pd.Series(result[0].analyzers._TimeReturn.get_analysis())
    ```

    除了上面提到的这些内置分析器外，Backtrader 当然还支持自定义分析器（不然就不符合 Backtrader style 了）。凡是涉及到自定义的操作，遵循的都是“在继承了 xxx 原始父类的基础上，在新的子类里自定义相关属性和方法”，比如《数据篇》中通过继承数据加载父类 bt.feeds.PandasData 等自定义数据加载函数、《指标篇》中在通过继承 bt.Indicator 自定义指标、《交易篇（上）》中通过继承 bt.CommInfoBase 自定义交易费用函数...... 不过，自定义分析器的过程与今天《策略篇》最开始介绍的定义策略函数是最相似的，分析器毕竟是用来分析整个回测的，既涉及过程，又涉及结果，所以继承的 bt.Analyzer 父类中的方法和相应的运行逻辑和策略中的基本一致：

    ```
    import backtrader as bt # 导入 Backtrader
    # 创建分析器
    class MyAnalyzer(bt.Analyzer):
    # 初始化参数：比如内置分析器支持设置的那些参数
    params = (
    (...,...), # 最后一个“,”最好别删！
    )
    # 初始化函数
    def __init__(self):
    '''初始化属性、计算指标等'''
    pass
    # analyzer与策略一样，都是从第0根bar开始运行
    # 都会面临 min_period 问题
    # 所以都会通过 prenext、nextstart 来等待 min_period 被满足
    def start(self):
    pass
    def prenext(self):
    pass
    def nextstart(self):
    pass
    def next(self):
    pass
    def stop(self):
    # 一般对策略整体的评价指标是在策略结束后开始计算的
    pass
    # 支持与策略一样的信息打印函数
    def notify_order(self, order):
    '''通知订单信息'''
    pass
    def notify_trade(self, trade):
    '''通知交易信息'''
    pass
    def notify_cashvalue(self, cash, value):
    '''通知当前资金和总资产'''
    pass
    def notify_fund(self, cash, value, fundvalue, shares):
    '''返回当前资金、总资产、基金价值、基金份额'''
    pass
    def get_analysis(self):
    pass
    # 官方提供的 SharpeRatio 例子
    class SharpeRatio(Analyzer):
    params = (('timeframe', TimeFrame.Years), ('riskfreerate', 0.01),)
    def __init__(self):
    super(SharpeRatio, self).__init__()
    self.anret = AnnualReturn()
    def start(self):
    # Not needed ... but could be used
    pass
    def next(self):
    # Not needed ... but could be used
    pass
    def stop(self):
    retfree = [self.p.riskfreerate] * len(self.anret.rets)
    retavg = average(list(map(operator.sub, self.anret.rets, retfree)))
    retdev = standarddev(self.anret.rets)
    self.ratio = retavg / retdev
    def get_analysis(self):
    return dict(sharperatio=self.ratio)
    ```

    下面是在 Backtrader 社区中找到的自定义分析器，用于查看每笔交易盈亏情况：

*   地址：https://community.backtrader.com/topic/1274/closed-trade-list-including-mfe-mae-analyzer；
*   该案例涉及到 trade 对象的相关属性，具体可以参考官方文档：https://www.backtrader.com/docu/trade/ 。

    ```
    class trade_list(bt.Analyzer):
    def __init__(self):
    self.trades = []
    self.cumprofit = 0.0
    def notify_trade(self, trade):
    if trade.isclosed:
    brokervalue = self.strategy.broker.getvalue()
    dir = 'short'
    if trade.history[0].event.size > 0: dir = 'long'
    pricein = trade.history[len(trade.history)-1].status.price
    priceout = trade.history[len(trade.history)-1].event.price
    datein = bt.num2date(trade.history[0].status.dt)
    dateout = bt.num2date(trade.history[len(trade.history)-1].status.dt)
    if trade.data._timeframe >= bt.TimeFrame.Days:
    datein = datein.date()
    dateout = dateout.date()
    pcntchange = 100 * priceout / pricein - 100
    pnl = trade.history[len(trade.history)-1].status.pnlcomm
    pnlpcnt = 100 * pnl / brokervalue
    barlen = trade.history[len(trade.history)-1].status.barlen
    pbar = pnl / barlen
    self.cumprofit += pnl
    size = value = 0.0
    for record in trade.history:
    if abs(size) < abs(record.status.size):
    size = record.status.size
    value = record.status.value
    highest_in_trade = max(trade.data.high.get(ago=0, size=barlen+1))
    lowest_in_trade = min(trade.data.low.get(ago=0, size=barlen+1))
    hp = 100 * (highest_in_trade - pricein) / pricein
    lp = 100 * (lowest_in_trade - pricein) / pricein
    if dir == 'long':
    mfe = hp
    mae = lp
    if dir == 'short':
    mfe = -lp
    mae = -hp
    self.trades.append({'ref': trade.ref,
    'ticker': trade.data._name,
    'dir': dir，
    'datein': datein,
    'pricein': pricein,
    'dateout': dateout,
    'priceout': priceout,
    'chng%': round(pcntchange, 2),
    'pnl': pnl, 'pnl%': round(pnlpcnt, 2),
    'size': size,
    'value': value,
    'cumpnl': self.cumprofit,
    'nbars': barlen, 'pnl/bar': round(pbar, 2),
    'mfe%': round(mfe, 2), 'mae%': round(mae, 2)})
    def get_analysis(self):
    return self.trades
    ```

    调用时，需要设置 cerebro.run(tradehistory=True)：

    ```
    # 添加自定义的分析指标
    cerebro.addanalyzer(trade_list, _name='tradelist')
    # 启动回测
    result = cerebro.run(tradehistory=True)
    # 返回结果
    ret = pd.DataFrame(result[0].analyzers.tradelist.get_analysis())
    # 部分结果展示
    ref ticker dir datein pricein dateout priceout chng% pnl pnl% size value cumpnl nbars pnl/bar mfe% mae%
    0 6586 000612.SZ long 2019-02-01 36.838173 2019-03-01 46.338544 25.79 116351.042167 0.10 12247 4.511571e+05 1.163510e+05 15 7756.74 29.74 0.00
    1 6587 000636.SZ long 2019-02-01 172.762500 2019-03-01 236.616875 36.96 329424.720625 0.28 5159 8.912817e+05 4.457758e+05 15 21961.65 52.94 0.00
    2 6591 000766.SZ long 2019-02-01 19.804062 2019-03-01 25.163577 27.06 93877.266141 0.08 17516 3.468879e+05 5.396530e+05 15 6258.48 30.20 -0.33
    3 6592 000807.SZ long 2019-02-01 23.945099 2019-03-01 31.359917 30.97 264345.664040 0.22 35651 8.536667e+05 8.039987e+05 15 17623.04 36.36 0.00
    4 6593 000829.SZ long 2019-02-01 69.728937 2019-03-01 90.499258 29.79 129939.131930 0.11 6256 4.362242e+05 9.339378e+05 15 8662.61 40.43 -0.64
    ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ...
    797 7390 600959.SH long 2020-11-02 4.809258 2021-01-04 4.495455 -6.52 -73971.303962 -0.05 235725 1.133662e+06 4.854122e+07 44 -1681.17 5.09 -7.94
    798 7445 601717.SH long 2020-12-01 23.003961 2021-01-04 25.823190 12.26 125700.963423 0.08 44587 1.025678e+06 4.866692e+07 23 5465.26 24.30 -4.33
    799 7448 603198.SH long 2020-12-01 36.751896 2021-01-04 41.230664 12.19 215057.006417 0.14 48017 1.764716e+06 4.888198e+07 23 9350.30 36.27 -5.69
    800 7310 603659.SH long 2020-08-03 105.138155 2021-01-04 116.205444 10.53 322492.512325 0.21 30301 3.185791e+06 4.920447e+07 103 3131.00 21.09 -11.13
    801 7395 603816.SH long 2020-11-02 109.871963 2021-01-04 106.540841 -3.03 -97628.521708 -0.06 29308 3.220127e+06 4.910684e+07 44 -2218.83 14.94 -8.55
    ```

    **如何对策略进行参数优化**

    如果策略的收益表现可能受相关参数的影响，需要验证比较参数不同取值对策略表现的影响，就可以使用 Backtrader 的参数优化功能，使用该功能只需通过 cerebro.optstrategy() 方法往大脑添加策略即可：

    ```
    class TestStrategy(bt.Strategy):
    params=(('period1',5),
    ('period2',10),) #全局设定均线周期
    ......
    # 实例化大脑
    cerebro1= bt.Cerebro(optdatas=True, optreturn=True)
    # 设置初始资金
    cerebro1.broker.set_cash(10000000)
    # 加载数据
    datafeed1 = bt.feeds.PandasData(dataname=data1, fromdate=datetime.datetime(2019,1,2), todate=datetime.datetime(2021,1,28))
    cerebro1.adddata(datafeed1, name='600466.SH')
    # 添加优化器
    cerebro1.optstrategy(TestStrategy, period1=range(5, 25, 5), period2=range(10, 41, 10))
    # 添加分析指标
    # 返回年初至年末的年度收益率
    cerebro1.addanalyzer(bt.analyzers.AnnualReturn, _name='_AnnualReturn')
    # 计算最大回撤相关指标
    cerebro1.addanalyzer(bt.analyzers.DrawDown, _name='_DrawDown')
    # 计算年化收益
    cerebro1.addanalyzer(bt.analyzers.Returns, _name='_Returns', tann=252)
    # 计算年化夏普比率
    cerebro1.addanalyzer(bt.analyzers.SharpeRatio_A, _name='_SharpeRatio_A')
    # 返回收益率时序
    cerebro1.addanalyzer(bt.analyzers.TimeReturn, _name='_TimeReturn')
    # 启动回测
    result = cerebro1.run()
    # 打印结果
    def get_my_analyzer(result):
    analyzer = {}
    # 返回参数
    analyzer['period1'] = result.params.period1
    analyzer['period2'] = result.params.period2
    # 提取年化收益
    analyzer['年化收益率'] = result.analyzers._Returns.get_analysis()['rnorm']
    analyzer['年化收益率（%）'] = result.analyzers._Returns.get_analysis()['rnorm100']
    # 提取最大回撤(习惯用负的做大回撤，所以加了负号)
    analyzer['最大回撤（%）'] = result.analyzers._DrawDown.get_analysis()['max']['drawdown'] * (-1)
    # 提取夏普比率
    analyzer['年化夏普比率'] = result.analyzers._SharpeRatio_A.get_analysis()['sharperatio']
    return analyzer
    ret = []
    for i in result:
    ret.append(get_my_analyzer(i[0]))
    pd.DataFrame(ret)
    # 优化结果
    period1 period2 年化收益率 年化收益率（%） 最大回撤（%） 年化夏普比率
    0 5 10 4.024514e-05 4.024514e-03 -0.010175 -140.948647
    1 5 20 -3.240455e-06 -3.240455e-04 -0.008839 -229.402157
    2 5 30 -1.211110e-05 -1.211110e-03 -0.008674 -236.577612
    3 5 40 -1.284502e-05 -1.284502e-03 -0.011886 -370.807650
    4 10 10 0.000000e+00 0.000000e+00 -0.000000 NaN
    5 10 20 8.568641e-06 8.568641e-04 -0.009392 -282.835125
    6 10 30 1.835459e-06 1.835459e-04 -0.008545 -265.568666
    7 10 40 -7.817367e-06 -7.817367e-04 -0.013492 -261.387903
    8 15 10 -6.560915e-09 -6.560915e-07 -0.017579 -161.893285
    9 15 20 -1.857955e-05 -1.857955e-03 -0.009652 -611.196458
    10 15 30 -2.226534e-05 -2.226534e-03 -0.008160 -641.959703
    11 15 40 1.708522e-05 1.708522e-03 -0.013492 -213.637841
    12 20 10 -3.799574e-05 -3.799574e-03 -0.025414 -109.665911
    13 20 20 0.000000e+00 0.000000e+00 -0.000000 NaN
    14 20 30 -1.398007e-05 -1.398007e-03 -0.010388 -527.518303
    15 20 40 6.699340e-06 6.699340e-04 -0.013492 -301.729232
    # 策略表现真的是惨不忍睹啊......
    ```

*   cerebro.optstrategy(strategy, *args, **kwargs)：strategy 就是自定义的策略类（比如上例的TestStrategy）、后面*args, **kwargs 对应自定义策略类中 params 中的需要优化的参数的取值（比如上例的period1=range(5, 25, 5), period2=range(10, 41, 10)）；当有多个参数时，会将各个参数的各个取值进行一一匹配（见上面的输出结果）；
*   在进行参数优化时，实例化大脑的时候，有 2 个与参数优化相关的参数：
*   optdatas=True：在处理数据时会采用相对节省时间的方式，进而提高优化速度；
*   optreturn=True：在返回回测结果时，为了节省时间，只返回与参数优化最相关的内容（params 和 analyzers），而不会返回参数优化不关心的数据（比如 datas, indicators, observers …等）；
*   参数优化是基于 multiprocessing 进行多进程处理数据和分析结果的。

*注意：在对于多个标的进行参数优化过程中（比如连续对1000个股票的均线策略寻优），如果对于多进程的cpu使用数量不加限制，会有一定几率出现异常错误的情况，这类错误目前还没找到解决方法。建议是限制cpu的数量，如设置为2或3： 

```
cerebro.run(maxcpus=2)
```

**总结**

一路学到现在，Backtrader 策略回测相关内容已经介绍的差不多了，大家可以总结一个属于自己的策略回测常规操作列表（操作框架），下面是公众号简单整理的，主要分“设置回测条件”、“编写交易策略”、“回测结果分析和评价”3 部分内容：

大家可以基于自己的策略操作列表回顾复习今天和之前的文章，来个知识串联。