+++
showonlyimage = true
draft = false
image = "img/portfolio/blog-2-trading-head.jpg"
date = "2023-05-19T5:46:00+05:30"
title = "4. Algorithm design powered by ChatGPT"
weight = 0
+++

Algorithmic Trading is allegedly fairly common, however almost kept behind lock and key for the majority of the non-technical public. Retail Traders appear to focus more on rather manual trading strategies. I love automation. So lets take a crack at it.
<!--more-->

*Again, disclaimer, this is catagorically not financial advice, do not replicate these steps, any risk you carry is all yours if you do.*

## Proposed Strategy

In order to be able to code the strategy, we first should spend some time to write down what it is as a matter of steps. This will make programming it much more straightforward, as well as maintaining a strong structure to the code.

The proposed Trading Strategy follows strict technical analysis, aiming for orders of RRR 1:3. This will become clear as to why it's important soon.

In terms of the analysis required for the algorithm to actually trigger a position, whether its a buy or a sell order, what we're looking for is price to arrive around a support or resistance zone, and then we'll be waiting for it to bounce.

Here's where the beauty of programming comes in. Along with the support/resistance zones, we can also check for trends where price appears to be forming an upward trend, or a downward trend. And then... we can backtest for what produces the highest probability of a successful trade with historical data, and go with that one! So many optimisations to be had in implementation! Since we'll have data on what is working and what isn't. Exciting!

## Support and Resistance Zones.

The code for analysis is something which is going to require some work, followed by some tuning. I dropped ChatGPT a query to get started:

`With mt5 and python, write a script to calculate support and resistance zones`

```python
import pandas as pd

def calculate_support_resistance(data, n):
    highs = data['High']
    lows = data['Low']
    closes = data['Close']
    pivot_points = []
    support_zones = []
    resistance_zones = []

    for i in range(n, len(data)):
        high_range = highs[i-n:i]
        low_range = lows[i-n:i]
        close_range = closes[i-n:i]
        pivot_point = (sum(high_range) + sum(low_range) + sum(close_range)) / (3 * n)
        pivot_points.append(pivot_point)
    
    for i in range(n, len(pivot_points)):
        if pivot_points[i] < min(pivot_points[i-n:i]):
            support_zones.append(pivot_points[i])
        elif pivot_points[i] > max(pivot_points[i-n:i]):
            resistance_zones.append(pivot_points[i])
    
    return support_zones, resistance_zones

# Example usage
# Assuming you have a DataFrame named 'data' with OHLC data
n_periods = 20  # Number of periods to calculate support and resistance
support, resistance = calculate_support_resistance(data, n_periods)

print("Support Zones:", support)
print("Resistance Zones:", resistance)
```

What is `OHLC Data`?? I hear you say. 

Open-High-Low-Close data. A candle consists of these 4 data points.

I then popped this into my script and rejigged a few names, and created a function to get the latest n number of candles.

`can you write a function to fetch the last 100 candles of data and input into a DataFrame named 'data' with OHLC data`

```python
import MetaTrader5 as mt5
import pandas as pd

def getLastCandles(symbol, n, timeframe):
    # Connect to MetaTrader 5
    mt5.initialize()

    # Request the last n candles of data
    candles = mt5.copy_rates_from_pos(symbol, timeframe, 0, n)

    # Create a DataFrame with OHLC data
    data = pd.DataFrame(candles, columns=['time', 'Open', 'High', 'Low', 'Close', 'Volume'])
    data['time'] = pd.to_datetime(data['time'], unit='s')

    # Close the connection to MetaTrader 5
    mt5.shutdown()

    return data

# Example usage
symbol = 'EURUSD'  # Replace with your desired symbol
n = 100  # Number of candles to fetch
timeframe = mt5.TIMEFRAME_D1  # Replace with your desired timeframe

data = getLastCandles(symbol, n, timeframe)
print(data)

```

This also required some rejigging. For example the line `data = pd.DataFrame(candles, columns=['time', 'Open', 'High', 'Low', 'Close', 'Volume'])` didn't quite work as well as was needed, so i searched up the example docs for that function and came across example code that didn't require the columns attribute.

Changed the function to be more like:

```python
def getLastCandles(symbol, n, timeframe):
    connectToMetaTrader()
    candles = mt5.copy_rates_from_pos(symbol, timeframe, 0, n)
    data = pd.DataFrame(candles)
    data['time'] = pd.to_datetime(data['time'], unit='s')
    disconnectFromMetaTrader()
    return data
```

This was then able to be passed into the above function for analysis, again, after the `highs = data['High']`, `lows = data['Low']` and `closes = data['Close']` was updated to remove the capital letter in the key. Looks like this is just the way mt5 gives us the data, with lowercase keys.

## First Result

So. I then had a bit of fun. Now we have an array of apparent zones which our first preliminary analysis algorithm thinks are support/resistance zones. I think this is going to be the most complex part of the analysis, once we have a good set of zones laid out, checking when price going past them and then hits again should be relatively straightforward. And so, the zones, we should definitely check if they're whats expected.

Enter: **[Matplotlib](https://matplotlib.org/stable/api/index).**

### Glorious Programming

The beauty of programming rather than doing things by hand - repeatability. Everytime you run a script for the same input data, it returns the same output. Well, not all code does this of course, anything with any level of randomisation incorporated, or nowerdays AI will not do this, however a script like this will. And there's beauty in that, because whereas when doing things by hand, you need to account for fluctuations in mood/attention/irritability and the likes of the person doing it, with code, to test different alterations of a strategy, you simple wack it in a for loop and try them out.

For example, the `calculate_support_resistance()` function takes in the parameters of `data`, and `n`. Now `n` here is the period to calculate over. Now whats the best period to chose? Who knows! So lets wack it in a loop, and iterate over a reasonable set of values, and then output the results for good measure and see what different values of `n` do!

![](../../img/portfolio/mt5-blog4-first-supp-res-output1.gif)

Beautiful.

*Note, this is the n value changed from 1 to 149. And.. none of them really look quite like what we're looking for! :D*

## Other cleanup done after the last update

1. Added the code for the script to Git for source control
2. Wacked everything into functions, so now we have a:
  `connectToMetaTrader()`, `disconnectFromMetaTrader()`, `openPosition(x,y,z)`, `closePosition(x)`, `getLastCandles(n)`, `calculateSupportResistance()` and `plotSupportResistanceZones()`.
  So I can do something like this now for the GIF above *(ignore the distinct lack of comments!)*:
  ```python
  candlesIn = getLastCandles(SYMBOL, 200, mt5.TIMEFRAME_H1)
  for i in range(1,150,1):
    supportZones, resistanceZones = calculateSupportResistance(candlesIn, i)
    print("N {}, Number of Zones {}".format(i, (len(supportZones) + len(resistanceZones))))
    plt = plotSupportResistanceZones(candlesIn, supportZones[:len(candlesIn)], resistanceZones[:len(candlesIn)])
    plt.savefig("figure{}.png".format(i), dpi=350)
    plt.close()
    print("Outputted.")
  ```
3. Renamed a few functions so the code is more readable

## Next steps:

1. Try another Support/Resistance Zone finding algorithm
2. Tune said algorithm
3. Create function for initiating trade based on break and retest
4. Working towards: First test run
5. Backtest to oblvion and optimise

~H 

_2023-05-19_
