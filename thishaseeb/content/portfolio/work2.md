+++
showonlyimage = true
draft = false
image = "img/portfolio/blog-2-trading-head.jpg"
date = "2023-05-08T23:39:00+05:30"
title = "Prelim MT5 + Python interface for Algo-trading"
weight = 0
+++

Algorithmic Trading is allegedly fairly common, however almost kept behind lock and key for the majority of the non-technical public. Retail Traders appear to focus more on rather manual trading strategies. I love automation. So lets take a crack at it.
<!--more-->

In the past I've come across a fair few Graduate roles and general Jobs centered around Data Science/Engineering for Automated trading algorithms. First off I think there's some contraints to be defined here. I'll be focusing my endevours with a focus on Forex and financial markets, as I know the most about these. They're the most liquid and highly volatile also, and i don't really enjoy fancy testing a longer term algorithm from the get go! 

Worth mentioning, this is just a blog, not financial advise, so don't take it as such, and don't repeat my steps. If you do so, you do it at your own risk.

## Statement of Intentions.

I intend on making this space a relatively unspecific catalogue of proejcts undergone. I had a few ideas of things to try that could be fun to document, and hence have created this space to do exactly that!

Hopefully the overhead in writing a post is not tooooo high, but I have attempted to automate what i can to reduce said overhead! Thank you hugo, and Github Pages.

I did also dabble with using Wordpress for this blog, but the page loadtimes would definitely take a hit with the advantage of ease of use, and the ability to write posts via a mobile device.

### Intended outcome

I desire to create an automated trading algorithm which can be run, and opens and closes positions on the financial markets based on it's own analyis. The strategy which I will be focusing on is a Break and Retest support/resistance of a pair such as GBP/USD. This is a common strategy used by Retail traders with a more technical trading influence, in this case, the algorithm will have no bearing on current events surrounding the market, and so would be considered purely technical.
### Background Knowledge

Two things which will help this endevour are:
1. Programming Skill
2. Trading Skill

### Programming Skill

I have a decent grasp on Python as the language of choice, having studied it formally for A Level, as well as using it in a professional environment as the scripting language of choice for anything that requires API access. I've also spend a good portion of time debugging production grade Flask web servers in a professional environment. I reckon i can pick up anything required in a reasonably quick amount of time.

### Trading Skill

I'm by no means a professional retail forex trader, however I have watched a fair few Trading Nut podcast (would recommend!), and completed the MissionFX trading course by Nick Shawn. I also did my fair share of practising a technical trading algorithm (the one I intend to automate here), inspired heavilly by the good Ser Nick Shawn's work, as well as taking some guidance from other traders on YouTube I've seen through the years. With my background in development, when doing the trading course, the thing that was always in the back of my mind was, "Man, surely this can be automated".

### Initial research

There are a plentiful supply of APIs available for accessing the Forex markets' historical and realtime data.

There is also an interface module for Metatrader 5 (MT5) for Python. 

I am fairly familiar with MT4, since I used it for backtesting the strategy taught by MissionFX. It also has a handy dandy mobile applicaion for monitoring open positions.

If we go down the API route, ideally an API which is created by a broker would be best, since we want to be making trades based on the same information as the analysis. IG markets' appears to be widely used. I also came across the [Currency Data API](https://apilayer.com/marketplace/currency_data-api?txn=free&live_demo=show) which looks promising and easy to use, however there is a cost attributed with both based on the number of requests. Visualising data from an API as well as currently open trades would be a task in itself. And so the option of MT5 seems more and more appealing.

### Next steps

1. Open up a broker account/demo account with a broker. Likely IC markets, as I'm most familiar with them, and they offer a spreadbetting account in the UK which has tax benefits.
2. Install Python3 and MT5 module
3. Write initial test script to have Python able to interact with MT5
4. Get trading data from MT5
5. Open test positions in Demo Account

~H 

_2023-05-08_
