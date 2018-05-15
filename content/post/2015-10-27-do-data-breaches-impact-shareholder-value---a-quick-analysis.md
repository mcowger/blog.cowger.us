---
layout: post
title: Do Data Breaches Impact Shareholder Value - A Quick Analysis
author: mcowger
categories:
  - data
  - security
  - breach
  - twitter
published: true
comments: true
---

I had an interesting chat on twitter today with a friend and colleague Wade O'Harrow.  He used to be at a bunch of places, but currently leads the SE team for HyTrust, a security company (yes, thats oversimplifying it).  He started the exhange with:  

>[Data is growing at exponential rates, and the trend of Corporate data breaches is staggering. still many question the need for protection.](https://twitter.com/wadeoharrow/status/659079598202732545)

and we eventually wandered into the discussion of what data breaches cost shareholders, with Wade making the suggestion that

> [@mcowger - The Risk vs. Reward question.  Productivity is great, but a security breach could wipe out more shareholder value long term.](https://twitter.com/wadeoharrow/status/659082294863507456)

I said as much on Twitter, but I'm not certain this is actually true.  Note - I'm not arguing that we should have security, or that the breaches aren't bad.   I'm arguing that they don't seem to impact long term share holder value.

So I decided to test this theory, and made the following assumptions / constraints

* The company being analyzed must be public, so we can see what happens with shareholder value.
* The company must not have undergone a major acquisition/merger/stock/new dividen/split event.
* The breach must have happened at least 1 year ago.
* We'd need a benchmark to make sure that any changes in the company's stock price was not simply a reflection of the market as  a whole.  I chose [Vanguard's VTSAX](https://personal.vanguard.com/us/funds/snapshot?FundId=0585&FundIntExt=INT) - Total Stock Market Index Fund as the benchmark.
* I'll compare the price of the stock (and benchmark) approx 2 weeks before public notification of the event (subject to weekdays/holidays) and exactly 1 year after that date.  The coincides with the [IRS definition of 'long term'](https://www.irs.gov/taxtopics/tc409.html) when it comes to investments.

Stock  | Event Notification  | Before Date  | Stock Price  | Benchmark Price  | 1 Year Date  | Stock Price  | Benchmark Price  | % Change Stock  | % Change Benchmark
--|---|---|---|---|---|---|---|---|--
Michaels  | April 14, 2014  | April 1, 2014  | 26.89  | 47.81  | April 1, 2015  | 22.36  | 52.09  | -16.85%  | +8.95%
UPS  | Aug 21, 2014  | Aug 1, 2014  | 97.03  | 48.41  | Aug 3, 2015  | 102.75  | 52.79  | +5.89%  | +9.04%
Home Depot  | Sept 18, 2014  | Sept 2, 2014  | 91.15  | 50.60  | Sept 2, 2015  | 116.48  | 49.21  | +27.78%%  | -2.74&%
KMART  | Oct 10, 2014  | Oct 1, 2014  | 17.57  | 48.62  | Oct 1, 2015  | 23.95  | 48.14  | +36.31%  | -0.98%
Target  | Dec 18, 2013  | Dec 2, 2013  | 62.73  | 45.58  | Dec 2, 2014  | 73.07  | 51.77  | +16.48%  | +13.58%

As is clear from the raw data above, in all cases but one (Michaels), the stock price (shareholder value) of the affected companies all went up.  In 3 out of the 5 cases (60%), we see that the benchmark is handily beat as well.

As such, I stand by my assertion that, today, massive data breaches may have notable costs for the public at large and for the company affected, but they don't appear to have a notable impact on long term shareholder value.
