+++
showonlyimage = true
draft = false
image = "img/portfolio/blog-2-trading-head.jpg"
date = "2023-05-12T23:30:00+05:30"
title = "3. Script for the first position"
weight = 0
+++

Algorithmic Trading is allegedly fairly common, however almost kept behind lock and key for the majority of the non-technical public. Retail Traders appear to focus more on rather manual trading strategies. I love automation. So lets take a crack at it.
<!--more-->

Following on from the last installment. The example code was failing to run successfully with the following error:
```
1. order_send(): by GBPUSD 0.1 lots at 1.245 with deviation=20 points
2. order_send failed, retcode=10030
   retcode=10030
   deal=0
   order=0
   volume=0.0
   price=0.0
   bid=0.0
   ask=0.0
   comment=Unsupported filling mode
   request_id=0
   retcode_external=0
   request=TradeRequest(action=1, magic=234000, order=0, symbol='GBPUSD', volume=0.1, price=1.245, stoplimit=0.0, sl=1.2440000000000002, tp=1.246, deviation=20, type=0, type_filling=2, type_time=0, expiration=0, comment='python script open', position=0, position_by=0)
```

So in the process we're still here!:

`3. Write initial test script to have Python able to interact with MT5`

## Debugging the Script

Looks like the error arises due to the defined filling moe being unsupported. Not really sure what that is. So lets give it a quick [ChatGPT](https://chat.openai.com/).

`what are the supported filing modes for the mt5 python module, for subitting orders`

```
The MetaTrader 5 (MT5) Python module provides several supported filing modes for submitting orders. These filing modes determine how orders are executed and processed by the trading platform. The available filing modes are:

ORDER_FILLING_FOK (Fill or Kill): This mode requires the entire order to be executed immediately and completely; otherwise, it will be canceled.

ORDER_FILLING_IOC (Immediate or Cancel): This mode executes the order partially or completely, but any portion that cannot be executed immediately will be canceled.

ORDER_FILLING_RETURN (Return): This mode allows partial order execution. Any portion of the order that cannot be executed immediately remains in the order book until it is executed or canceled manually.

ORDER_FILLING_RESERVED (Reserved): This mode is used for internal purposes and is not available for regular trading operations.

When submitting an order using the MT5 Python module, you need to specify the desired filing mode as a parameter in the order request. For example, if you want to submit an order with Fill or Kill mode, you would set the filing mode as ORDER_FILLING_FOK in the order request.

Please note that the availability and behavior of these filing modes may vary depending on the specific broker and trading environment you are using with the MT5 platform. It's always a good idea to consult the documentation or API reference of your broker or the MT5 Python module for the most accurate and up-to-date information regarding filing modes and their usage.
```

So, lets try changing the line

`mt5.ORDER_FILLING_RETURN`

to 

`mt5.ORDER_FILLING_IOC` 

instead.

**Progress!**

Order opened and Closed Successfully. Woo!

```
python3 .\exampleconnect.py
MetaTrader5 package author:  MetaQuotes Ltd.
MetaTrader5 package version:  5.0.45
1. order_send(): by GBPUSD 0.1 lots at 1.245 with deviation=20 points
2. order_send done,  OrderSendResult(retcode=10009, deal=285100830, order=394540683, volume=0.1, price=1.245, bid=1.24491, ask=1.245, comment='Request executed', request_id=1157830844, retcode_external=0, request=TradeRequest(action=1, magic=234000, order=0, symbol='GBPUSD', volume=0.1, price=1.245, stoplimit=0.0, sl=1.2440000000000002, tp=1.246, deviation=20, type=0, type_filling=1, type_time=0, expiration=0, comment='python script open', position=0, position_by=0))
   opened position with POSITION_TICKET=394540683
   sleep 2 seconds before closing position #394540683
3. close position #394540683: sell GBPUSD 0.1 lots at 1.24491 with deviation=20 points
4. position #394540683 closed, OrderSendResult(retcode=10009, deal=285100831, order=394540686, volume=0.1, price=1.24491, bid=1.24491, ask=1.245, comment='Request executed', request_id=1157830845, retcode_external=0, request=TradeRequest(action=1, magic=234000, order=0, symbol='GBPUSD', volume=0.1, price=1.24491, stoplimit=0.0, sl=0.0, tp=0.0, deviation=20, type=1, type_filling=1, type_time=0, expiration=0, comment='python script close', position=394540683, position_by=0))
   retcode=10009
   deal=285100831
   order=394540686
   volume=0.1
   price=1.24491
   bid=1.24491
   ask=1.245
   comment=Request executed
   request_id=1157830845
   retcode_external=0
   request=TradeRequest(action=1, magic=234000, order=0, symbol='GBPUSD', volume=0.1, price=1.24491, stoplimit=0.0, sl=0.0, tp=0.0, deviation=20, type=1, type_filling=1, type_time=0, expiration=0, comment='python script close', position=394540683, position_by=0)
       traderequest: action=1
       traderequest: magic=234000
       traderequest: order=0
       traderequest: symbol=GBPUSD
       traderequest: volume=0.1
       traderequest: price=1.24491
       traderequest: stoplimit=0.0
       traderequest: sl=0.0
       traderequest: tp=0.0
       traderequest: deviation=20
       traderequest: type=1
       traderequest: type_filling=1
       traderequest: type_time=0
       traderequest: expiration=0
       traderequest: comment=python script close
       traderequest: position=394540683
       traderequest: position_by=0
```

# Cleaning up Code

Cleaning up code is always good, to make is more adaptable, readable, and generally put you in a good spot for debugging.

I haven't added comments here but have added some print statements and seperated the example code into functions to be called instead of a single sequence of steps.

This here would be an example of the adapted `openPosition` function. The final program will likely operate an analysis within a `while True:` loop, making it constantly reassess the algorithm and goods like that.

```python
def openPosition(symbol, price):
    connectToMetaTrader()
    if SENDING_ORDER:
        return
    SENDING_ORDER = True
    request = {
        "action": mt5.TRADE_ACTION_DEAL,
        "symbol": symbol,
        "volume": lot,
        "type": mt5.ORDER_TYPE_BUY,
        "price": price,
        "sl": price - 100 * point,
        "tp": price + 100 * point,
        "deviation": DEVIATION,
        "magic": 17829,
        "comment": "Open Order",
        "type_time": mt5.ORDER_TIME_GTC,
        "type_filling": mt5.ORDER_FILLING_IOC,
    }
    ORDERS.append(mt5.order_send(request))

    print("[info] openPosition |Sent Order:\n\tSymbol: {} \n\tLots: {} \n\tPrice: {}".format(symbol,lot,price));

    if ORDERS[-1].retcode != mt5.TRADE_RETCODE_DONE:
        print("[error] openPosition | order_send failed, retcode={}".format(ORDERS[-1].retcode))
        result_dict=result._asdict()
        for field in result_dict.keys():
            print("   {}={}".format(field,result_dict[field]))
            if field=="request":
                traderequest_dict=result_dict[field]._asdict()
                for tradereq_filed in traderequest_dict:
                    print("[error] openPosition | traderequest: {}={}".format(tradereq_filed,traderequest_dict[tradereq_filed]))
        print("[info] openPosition | shutdown() and quit")
        disconnectFromMetaTrader()
    SENDING_ORDER = False
    print("[info] openPosition | order_send done, ", ORDERS[-1])
    print("[info] openPosition |  opened position with POSITION_TICKET={}".format(ORDERS[-1].order))
    print("[info] openPosition |  sleep 2 seconds before closing position #{}".format(ORDERS[-1].order))
```

## Creating a Git Repository

Adding version control to your code gives a lot of benefits, even if not pushed to a remote repository. The ability to see changes in code between intervals of progress can give much benefit. In my case, I have a github account, and I already have access set up via an ssh key, and so pushing to a new remote repository is straight forward, so I've done it. 

Checkout my public repos and contributions at [Github - thishaseeb](https://github.com/thishaseeb).

*Worth noting, my professinal work is not under this user, so it's not an accurate representation of the work I've done over the last few years!*

## Next steps:

**Algorithm Design.**

It's time to get the strategy on paper in diagram form and in a manner which will make it straight forwards, or reasonable straight forward to put into code! 

~H 

_2023-05-12_
