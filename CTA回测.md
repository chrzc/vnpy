# backtesting

## class OptimizationSetting
* 用于优化的基类
* __init__(self)初始化参数
    params = {} , target_name = ""
*  add_parameter(
        self, name: str, start: float, end: float = None, step: float = None
    ):
![image.png](https://upload-images.jianshu.io/upload_images/2792688-d1e9439c4caf8e67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    * 如图 构建成
        [start,...,end]间隔为step的列表,作为name对应的值,更新params字典
        
* set_target(self, target_name: str):
* generate_setting() 
迭代params 利用itertools.product包对params.values()求笛卡尔积,构建为如图的输出
![image.png](https://upload-images.jianshu.io/upload_images/2792688-41a54ce494482af1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* generate_setting_ga()
和generate_setting类似,一样迭代params,只是改变了输出,如图
![image.png](https://upload-images.jianshu.io/upload_images/2792688-22a9ba4ef4fc1760.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## BacktestingEngine
* __init__(self)
* clear_data() 重新初始化所有数据
* set_parameters(
    self,
    vt_symbol: str,
    interval: Interval,
    start: datetime,
    rate: float,
    slippage: float,
    size: float,
    pricetick: float,
    capital: int = 0,
    end: datetime = None,
    mode: BacktestingMode = BacktestingMode.BAR,
    inverse: bool = False
    )
    设置BacktestingEngine的参数
* add_strategy(self, strategy_class: type, setting: dict)
strategy_class是以CtaTemplate为基类的策略类,可以由自己定义.
加载strategy_class,并初始化vt_symbol,setting,更新BacktestingEngine.strategy_class和strategy
* load_data()
清空self.history_data,在self.start和self.end之间循环执行如下(tick类似)
```
    @abstractmethod
    def load_bar_data(
        self,
        symbol: str,
        exchange: "Exchange",
        interval: "Interval",
        start: datetime,
        end: datetime
    ) -> Sequence["BarData"]:
        pass
```
每次循环history_data.extend()

TO_DO

* cross_limit_order(self):
撮合限价单委托
    * 通过判断限价与当前价，调用策略的on_order、on_trade函数，并更新仓位
    * 遍历订单状态是"not traded"的订单,将状态是"提交中"的改为"未成交",
    *              
        ```
        long_cross = (
            order.direction == Direction.LONG
            and order.price >= long_cross_price
            and long_cross_price > 0
        )
        ```
        * 目的是要低于限价买入，long_cross_price 等于bar的最低价，bar的最低价小于设定的限价，所以要买入。
    * 
        ```
        short_cross = (
            order.direction == Direction.SHORT
            and order.price <= short_cross_price
            and short_cross_price > 0
        )
        ```
        * 目的是高于限价卖出
        ![image.png](https://upload-images.jianshu.io/upload_images/2792688-93505265667f38e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    *
        判断是否成交,continue;Push order 并订单状态改成"全部交易" 
    * **调用策略的on_order()函数 委托回报**,在策略中可以直接pass，其具体逻辑应用交给回测/实盘引擎负责,active_limit_orders pop这条order
    * 更新交易次数,记录交易价格(比较开盘价和限价),poschange
    ```
    if long_cross:
        trade_price = min(order.price, long_best_price)
        pos_change = order.volume
    else:
        trade_price = max(order.price, short_best_price)
        pos_change = -order.volume
    ```
    * **调用策略的on_trade()函数 成交回报** 
    * 直到active_limit_orders为空

        * active_limit_orders由以下构建 
        ```
            def send_limit_order(
                self,
                direction: Direction,
                offset: Offset,
                price: float,
                volume: float
            ):
                """"""
                self.limit_order_count += 1

                order = OrderData(
                    symbol=self.symbol,
                    exchange=self.exchange,
                    orderid=str(self.limit_order_count),
                    direction=direction,
                    offset=offset,
                    price=price,
                    volume=volume,
                    status=Status.SUBMITTING,
                    gateway_name=self.gateway_name,
                    datetime=self.datetime
                )

                self.active_limit_orders[order.vt_orderid] = order
                self.limit_orders[order.vt_orderid] = order

                return order.vt_orderid
        ```


* cross_stop_order  
撮合本地停止单
    * 和limit类似 增加了将order存入了limit_orders字典
    ```
    self.limit_orders[order.vt_orderid] = order
    ```
    * 增加了 Update stop order
    * 增加了执行策略的停止单回报on_stop_order()
    * orders由send_stop_order构建





* run_backtesting(self)

pass



* new_bar(self, bar: BarData)
调用trader/object/BarData类



## DailyResult












# engine

TODO 寻找为什么需要多线程 