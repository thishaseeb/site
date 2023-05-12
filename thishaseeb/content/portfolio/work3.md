+++
showonlyimage = true
draft = false
image = "img/portfolio/blog-2-trading-head.jpg"
date = "2023-05-11T23:39:00+05:30"
title = "2. Connecting to MT5 and creating helper functions"
weight = 0
+++

Algorithmic Trading is allegedly fairly common, however almost kept behind lock and key for the majority of the non-technical public. Retail Traders appear to focus more on rather manual trading strategies. I love automation. So lets take a crack at it.
<!--more-->

Okay, so. Following on from the last stage, this is the rough outline for what I figured would be next.

1. Open up a broker account/demo account with a broker. Likely IC markets, as I'm most familiar with them, and they offer a spreadbetting account in the UK which has tax benefits.
2. Install Python3 and MT5 module
3. Write initial test script to have Python able to interact with MT5
4. Get trading data from MT5
5. Open test positions in Demo Account

Lets try it.

*Note, I'm not giving any financial advise, and am not telling you to follow these steps, this is purely a blog for the purpose of entertainment and tracking. If you choose to repeat the steps, you choose to do so at your own risk.*

## Opening a trading account.

I already have one.
Done ✔.

## Install MT5 and Set up Account.

Before that we need to install MT5.

IC Markets has their own version, once you make an account, theres a link to download in the downloads section.

IC Markets also gives easy access to practise accounts, using paper money. Although depending on who's analysis of the world you agree with, all money could be considered paper money.

So. I've gone and downloaded and installed MT5 on my trusty steed my Asus laptop running yee ol Windows 11, and created a Demo account, and logged in.

Specs to keep is realistic:
```
Trading Platform: MT5
Account Type: Standard Account
Currency: GBP
Leverage: 1:30
Initial Deposit: 200
```

Of course, since we'll only be trialing the strat for the time being, the initial deposit doesn't matter so much. One we have the mechanics worked out thats when it will make the biggest difference.

Logs in. Oh boy, I forgot about Metatrader's glorious sound effects.

## Install Python3 and MT5 module

Maybe at this point its worth mentioning, this blog likely isn't the best to learn how to program from. I'll be approaching it as more of a log rather than a tutorial for beginners, for example if you're new to Python and don't know how to install it, there's probably a better guide out there on how to do that! YouTube is great also.

Standard stuff Really.

[Python Downloads Page](https://www.python.org/downloads/)

### My Version of events

The Software versions I'm running are as follows:

I'm using `WSL 2` running `Ubuntu 20.04.4 LTS`, `Python 3.8.10`, and `pip 20.0.2`.

### Installing MT5 module

I'm following [this](https://www.youtube.com/watch?v=zu2q28h9Uvc) guide for this process.

Oh. Metatrader's module doesn't support ubuntu. Guess we're working with powershell and windows.

`pip install MetaTrader5`

*fun fact*

If you have Python2 and Python3 installed, sometimes your pip command will point to the Python2 pip, and if you're developing with Python3, you'll find your modules that you're attempting to install.. won't. In that case what you can do is specify it as a module for Python3. I used this.

`python3 -m pip install MetaTrader5`

## Write initial test script to have Python able to interact with MT5

Looks like the MT5 Module has some example code.

```python
import time
import MetaTrader5 as mt5
 
# display data on the MetaTrader 5 package
print("MetaTrader5 package author: ", mt5.__author__)
print("MetaTrader5 package version: ", mt5.__version__)
 
# establish connection to the MetaTrader 5 terminal
if not mt5.initialize():
    print("initialize() failed, error code =",mt5.last_error())
    quit()
 
# prepare the buy request structure
symbol = "USDJPY"
symbol_info = mt5.symbol_info(symbol)
if symbol_info is None:
    print(symbol, "not found, can not call order_check()")
    mt5.shutdown()
    quit()
 
# if the symbol is unavailable in MarketWatch, add it
if not symbol_info.visible:
    print(symbol, "is not visible, trying to switch on")
    if not mt5.symbol_select(symbol,True):
        print("symbol_select({}}) failed, exit",symbol)
        mt5.shutdown()
        quit()
 
lot = 0.1
point = mt5.symbol_info(symbol).point
price = mt5.symbol_info_tick(symbol).ask
deviation = 20
request = {
    "action": mt5.TRADE_ACTION_DEAL,
    "symbol": symbol,
    "volume": lot,
    "type": mt5.ORDER_TYPE_BUY,
    "price": price,
    "sl": price - 100 * point,
    "tp": price + 100 * point,
    "deviation": deviation,
    "magic": 234000,
    "comment": "python script open",
    "type_time": mt5.ORDER_TIME_GTC,
    "type_filling": mt5.ORDER_FILLING_RETURN,
}
 
# send a trading request
result = mt5.order_send(request)
# check the execution result
print("1. order_send(): by {} {} lots at {} with deviation={} points".format(symbol,lot,price,deviation));
if result.retcode != mt5.TRADE_RETCODE_DONE:
    print("2. order_send failed, retcode={}".format(result.retcode))
    # request the result as a dictionary and display it element by element
    result_dict=result._asdict()
    for field in result_dict.keys():
        print("   {}={}".format(field,result_dict[field]))
        # if this is a trading request structure, display it element by element as well
        if field=="request":
            traderequest_dict=result_dict[field]._asdict()
            for tradereq_filed in traderequest_dict:
                print("       traderequest: {}={}".format(tradereq_filed,traderequest_dict[tradereq_filed]))
    print("shutdown() and quit")
    mt5.shutdown()
    quit()
 
print("2. order_send done, ", result)
print("   opened position with POSITION_TICKET={}".format(result.order))
print("   sleep 2 seconds before closing position #{}".format(result.order))
time.sleep(2)
# create a close request
position_id=result.order
price=mt5.symbol_info_tick(symbol).bid
deviation=20
request={
    "action": mt5.TRADE_ACTION_DEAL,
    "symbol": symbol,
    "volume": lot,
    "type": mt5.ORDER_TYPE_SELL,
    "position": position_id,
    "price": price,
    "deviation": deviation,
    "magic": 234000,
    "comment": "python script close",
    "type_time": mt5.ORDER_TIME_GTC,
    "type_filling": mt5.ORDER_FILLING_RETURN,
}
# send a trading request
result=mt5.order_send(request)
# check the execution result
print("3. close position #{}: sell {} {} lots at {} with deviation={} points".format(position_id,symbol,lot,price,deviation));
if result.retcode != mt5.TRADE_RETCODE_DONE:
    print("4. order_send failed, retcode={}".format(result.retcode))
    print("   result",result)
else:
    print("4. position #{} closed, {}".format(position_id,result))
    # request the result as a dictionary and display it element by element
    result_dict=result._asdict()
    for field in result_dict.keys():
        print("   {}={}".format(field,result_dict[field]))
        # if this is a trading request structure, display it element by element as well
        if field=="request":
            traderequest_dict=result_dict[field]._asdict()
            for tradereq_filed in traderequest_dict:
                print("       traderequest: {}={}".format(tradereq_filed,traderequest_dict[tradereq_filed]))
 
# shut down connection to the MetaTrader 5 terminal
mt5.shutdown()
 
```
[source](https://www.mql5.com/en/docs/python_metatrader5/mt5ordersend_py)

### Next steps

Make the example script work, since it errors at the moment.

~H 

_2023-05-11_
